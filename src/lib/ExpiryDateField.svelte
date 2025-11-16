<script lang="ts">
  // Svelte 5のprops構文を使用
  interface Props {
    form: any
  }

  const { form }: Props = $props()

  // フォーマット関数
  const formatExpiryDate = (
    previousValue: string,
    currentValue: string
  ): string => {
    // 数字のみを取得
    const numbersOnly = currentValue.replace(/\D/g, '')

    // 前の値の数字も取得
    const previousNumbers = previousValue.replace(/\D/g, '')

    // 削除操作かどうかを判定
    const isDeleting = numbersOnly.length < previousNumbers.length

    // 1桁で2-9の場合は自動で0埋めしてスラッシュを追加
    if (numbersOnly.length === 1 && !isDeleting) {
      const firstDigit = parseInt(numbersOnly)
      if (firstDigit >= 2 && firstDigit <= 9) {
        return `0${numbersOnly}/`
      }
    }

    // 2桁入力された場合、自動でスラッシュを追加
    if (numbersOnly.length >= 2) {
      let formatted = numbersOnly.substring(0, 2)
      if (numbersOnly.length > 2) {
        // 4桁を超えてもそのまま表示（バリデーションでエラーになる）
        formatted += '/' + numbersOnly.substring(2)
      } else if (!isDeleting) {
        // 2桁ちょうどで追加入力の場合のみスラッシュを追加
        formatted += '/'
      }
      return formatted
    }

    return numbersOnly
  }

  // バリデーション関数（入力中用）
  const validateExpiryDateOnChange = ({ value }: { value: string }) => {
    if (!value) {
      return undefined // 入力中は必須エラーを出さない
    }

    const numbersOnly = value.replace(/\D/g, '')

    if (numbersOnly.length === 0) {
      return '数字を入力してください'
    }

    // 4桁を超えた場合はエラー
    if (numbersOnly.length > 4) {
      return 'MM/YY形式（4桁の数字）で入力してください'
    }

    // 2桁以上の場合は月をチェック
    if (numbersOnly.length >= 2) {
      const month = parseInt(numbersOnly.substring(0, 2))
      if (month < 1 || month > 12) {
        return '正しい月を入力してください（01-12）'
      }
    }

    // 4桁入力完了時は年もチェック
    if (numbersOnly.length === 4) {
      const month = parseInt(numbersOnly.substring(0, 2))
      const year = parseInt(numbersOnly.substring(2, 4)) + 2000

      const currentDate = new Date()
      const currentYear = currentDate.getFullYear()
      const currentMonth = currentDate.getMonth() + 1

      if (
        year < currentYear ||
        (year === currentYear && month < currentMonth)
      ) {
        return '有効期限が過去の日付です'
      }
    }

    return undefined
  }

  // バリデーション関数（フォーカスアウト時用）
  const validateExpiryDateOnBlur = ({ value }: { value: string }) => {
    if (!value) {
      return '有効期限は必須です'
    }

    const numbersOnly = value.replace(/\D/g, '')

    if (numbersOnly.length === 0) {
      return '数字を入力してください'
    }

    if (numbersOnly.length !== 4) {
      return 'MM/YY形式（4桁の数字）で入力してください'
    }

    const month = parseInt(numbersOnly.substring(0, 2))
    const year = parseInt(numbersOnly.substring(2, 4)) + 2000

    if (month < 1 || month > 12) {
      return '正しい月を入力してください（01-12）'
    }

    const currentDate = new Date()
    const currentYear = currentDate.getFullYear()
    const currentMonth = currentDate.getMonth() + 1

    if (year < currentYear || (year === currentYear && month < currentMonth)) {
      return '有効期限が過去の日付です'
    }

    return undefined
  }

  // カスタムイベントハンドラー
  const handleInputChange = (fieldApi: any, event: Event) => {
    const target = event.target as HTMLInputElement
    const inputValue = target.value
    const previousValue = fieldApi.state.value || ''

    // フォーマット処理
    const formattedValue = formatExpiryDate(previousValue, inputValue)

    // 値を更新
    fieldApi.handleChange(formattedValue)

    // バリデーション実行
    const error = validateExpiryDateOnChange({ value: formattedValue })

    // エラー状態を設定
    fieldApi.setMeta((prev: any) => ({
      ...prev,
      errorMap: {
        ...prev.errorMap,
        onChange: error,
        onBlur: error,
      },
    }))
  }

  const handleInputBlur = (fieldApi: any, event: Event) => {
    const target = event.target as HTMLInputElement
    const value = target.value

    // blurバリデーション実行
    const error = validateExpiryDateOnBlur({ value })

    // エラー状態を設定
    fieldApi.setMeta((prev: any) => ({
      ...prev,
      isTouched: true,
      errorMap: {
        ...prev.errorMap,
        onBlur: error,
      },
    }))
  }
</script>

<form.Field name="expiryDate">
  {#snippet children(field: any)}
    <div class="mb-4">
      <label
        for="expiryDate"
        class="block text-sm font-medium text-gray-700 mb-2"
      >
        有効期限:
      </label>
      <input
        id="expiryDate"
        name={field.name}
        type="text"
        inputmode="numeric"
        autocomplete="cc-exp"
        pattern="[0-9/]*"
        value={field.state.value}
        onblur={e => handleInputBlur(field, e)}
        oninput={e => handleInputChange(field, e)}
        required
        class="w-full px-3 py-2 border rounded-md
               focus:outline-none focus:border-blue-500
               font-mono
               {field.state.meta.errors.length > 0
          ? 'border-red-500'
          : 'border-gray-300'}"
      />
      <p class="mt-1 text-xs text-gray-500">形式: MM/YY</p>
      {#if field.state.meta.errors.length > 0}
        <p class="mt-1 text-sm text-red-600" role="alert">
          {field.state.meta.errors[0]}
        </p>
      {/if}
    </div>
  {/snippet}
</form.Field>
