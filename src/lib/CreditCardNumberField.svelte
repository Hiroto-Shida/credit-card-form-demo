<script lang="ts">
  // Svelte 5のprops構文を使用
  interface Props {
    form: any
  }

  const { form }: Props = $props()

  // クレジットカードブランドの定義
  interface CardBrand {
    name: string
    maxLength: number
    pattern: RegExp
    formatPattern: number[] // フォーマットパターン（各グループの桁数）
  }

  const cardBrands: CardBrand[] = [
    {
      name: 'Visa',
      maxLength: 19,
      pattern: /^4/,
      formatPattern: [4, 4, 4, 4, 3], // 16-19桁対応
    },
    {
      name: 'Master Card',
      maxLength: 16,
      pattern: /^[25]/,
      formatPattern: [4, 4, 4, 4],
    },
    {
      name: 'JCB',
      maxLength: 16,
      pattern: /^35/,
      formatPattern: [4, 4, 4, 4],
    },
    {
      name: 'American Express',
      maxLength: 15,
      pattern: /^3[47]/,
      formatPattern: [4, 6, 5],
    },
    {
      name: 'Diners Club',
      maxLength: 14,
      pattern: /^36/,
      formatPattern: [4, 6, 4],
    },
  ]

  // ブランド判定関数
  const detectCardBrand = (cardNumber: string): CardBrand | null => {
    const numbers = cardNumber.replace(/\D/g, '')
    if (numbers.length === 0) return null

    for (const brand of cardBrands) {
      if (brand.pattern.test(numbers)) {
        return brand
      }
    }
    return null
  }

  // ブランド別フォーマット関数
  const formatByBrand = (numbers: string, brand: CardBrand | null): string => {
    if (!brand) {
      // ブランド不明の場合は4桁区切り
      return numbers.replace(/(\d{4})(?=\d)/g, '$1 ')
    }

    let formatted = ''
    let pos = 0

    for (let i = 0; i < brand.formatPattern.length; i++) {
      const groupLength = brand.formatPattern[i]
      const group = numbers.slice(pos, pos + groupLength)

      if (group.length === 0) break

      if (formatted.length > 0) {
        formatted += ' '
      }
      formatted += group
      pos += groupLength

      if (pos >= numbers.length) break
    }

    return formatted
  }

  // カード番号フォーマット関数 - 桁数制限なし版
  const formatCardNumber = (
    previousValue: string,
    currentValue: string
  ): { formatted: string; wasTruncated: boolean; brand: CardBrand | null } => {
    // 現在の値から数字のみを取得
    const numbersOnly = currentValue.replace(/\D/g, '')

    // ブランド判定（最初の数字で判定）
    const brand = detectCardBrand(numbersOnly)

    // 桁数制限は行わない（フォーマットのみ）
    const truncated = numbersOnly
    const wasTruncated = false

    // 前の値から数字のみを取得
    const previousNumbers = previousValue.replace(/\D/g, '')

    // 入力が追加されたかどうかを判定
    const isAdding = truncated.length > previousNumbers.length

    let formatted = ''

    // ブランド固有の桁数以内かどうかで処理を分岐
    if (brand && truncated.length <= brand.maxLength) {
      // ブランド固有のフォーマット
      formatted = formatByBrand(truncated, brand)

      // ブランド固有の末尾スペース追加処理
      if (isAdding) {
        let shouldAddSpace = false
        let currentPos = 0

        for (let i = 0; i < brand.formatPattern.length; i++) {
          currentPos += brand.formatPattern[i]

          if (
            truncated.length === currentPos &&
            i < brand.formatPattern.length - 1
          ) {
            // VISAの16桁目（標準的な桁数）の場合はスペースを追加しない
            if (brand.name === 'Visa' && truncated.length === 16) {
              shouldAddSpace = false
            } else {
              shouldAddSpace = true
            }
            break
          }
        }

        if (shouldAddSpace) {
          formatted = formatted + ' '
        }
      }
    } else if (brand && truncated.length > brand.maxLength) {
      // ブランド桁数超過の場合：既存フォーマット + 4桁区切り
      const brandPortionNumbers = truncated.slice(0, brand.maxLength)
      const excessNumbers = truncated.slice(brand.maxLength)

      // ブランド固有部分をフォーマット
      const brandFormatted = formatByBrand(brandPortionNumbers, brand)

      // 超過部分を4桁区切りでフォーマット
      const excessFormatted = excessNumbers.replace(/(\d{4})(?=\d)/g, '$1 ')

      // 結合
      formatted = brandFormatted
      if (excessNumbers.length > 0) {
        // VISAの17桁目（最初の超過桁）の場合は、16桁の後にスペースを追加
        if (brand.name === 'Visa' && excessNumbers.length === 1) {
          formatted += ' ' + excessNumbers
        } else {
          formatted += ' ' + excessFormatted
        }
      }

      // 超過部分の末尾スペース追加処理
      if (isAdding && excessNumbers.length > 0) {
        const shouldAddSpace =
          excessNumbers.length % 4 === 0 && excessNumbers.length > 1
        if (shouldAddSpace) {
          formatted = formatted + ' '
        }
      }
    } else {
      // ブランド不明の場合は4桁区切り
      formatted = truncated.replace(/(\d{4})(?=\d)/g, '$1 ')

      // 4桁区切りの末尾スペース追加処理
      if (isAdding) {
        const shouldAddSpace =
          truncated.length % 4 === 0 &&
          truncated.length > 0 &&
          truncated.length !== 16 // 16桁目ではスペースを追加しない

        if (shouldAddSpace) {
          formatted = formatted + ' '
        }
      }
    }

    return { formatted, wasTruncated, brand }
  }

  // 数字のみを取得（バリデーション用）
  const getNumbersOnly = (value: string): string => {
    return value.replace(/\D/g, '')
  }

  // Luhnアルゴリズムによる番号チェック
  const validateLuhn = (cardNumber: string): boolean => {
    const digits = cardNumber.replace(/\D/g, '').split('').map(Number)
    let sum = 0
    let isEven = false

    // 右から左へ処理
    for (let i = digits.length - 1; i >= 0; i--) {
      let digit = digits[i]

      if (isEven) {
        digit *= 2
        if (digit > 9) {
          digit -= 9
        }
      }

      sum += digit
      isEven = !isEven
    }

    return sum % 10 === 0
  }

  // ブランド別の最低桁数取得
  const getMinimumLength = (brand: CardBrand | null): number => {
    if (!brand) return 16 // ブランド不明の場合は16桁以上でチェック

    switch (brand.name) {
      case 'American Express':
        return 15
      case 'Diners Club':
        return 14
      case 'Visa':
      case 'Master Card':
      case 'JCB':
        return 16
      default:
        return 16
    }
  }

  // バリデーション関数 - ブランド対応版
  const validateCardNumber = (
    value: string,
    context: 'change' | 'blur',
    brand: CardBrand | null
  ) => {
    if (!value) {
      return context === 'change' ? undefined : 'クレジットカード番号は必須です'
    }

    const numbersOnly = getNumbersOnly(value)

    // 数字以外の文字が含まれている場合のチェック
    if (value.replace(/[\d\s]/g, '').length > 0) {
      return '数字のみで入力してください'
    }

    if (numbersOnly.length === 0) {
      return '数字を入力してください'
    }

    // Luhnアルゴリズムチェック（最低桁数以上の場合）
    const minimumLength = getMinimumLength(brand)
    if (numbersOnly.length >= minimumLength) {
      if (!validateLuhn(numbersOnly)) {
        return '存在しない番号です(Luhnアルゴリズム)'
      }
    }

    // ブランド別バリデーション
    if (brand) {
      // ブランド固有の最大桁数チェック
      if (numbersOnly.length > brand.maxLength) {
        return `${brand.name}は${brand.maxLength}桁以下で入力してください`
      }

      // ブランド固有の必要桁数チェック（blur時のみ）
      if (context === 'blur' && numbersOnly.length < brand.maxLength) {
        // American Expressは15桁、Diners Clubは14桁が正確な桁数
        if (brand.name === 'American Express' && numbersOnly.length !== 15) {
          return 'American Expressは15桁で入力してください'
        }
        if (brand.name === 'Diners Club' && numbersOnly.length !== 14) {
          return 'Diners Clubは14桁で入力してください'
        }
        // その他のブランドは最大桁数が必要桁数
        if (
          (brand.name === 'Visa' && numbersOnly.length < 16) ||
          (brand.name === 'Master Card' && numbersOnly.length < 16) ||
          (brand.name === 'JCB' && numbersOnly.length < 16)
        ) {
          return `${brand.name}は16桁で入力してください`
        }
      }
    } else {
      // ブランド不明の場合は汎用バリデーション
      if (numbersOnly.length > 19) {
        return '19桁以下で入力してください'
      }

      if (context === 'blur' && numbersOnly.length < 13) {
        return '13桁以上で入力してください'
      }
    }

    return undefined
  }

  // 直接的なイベントハンドラーでバリデーションを制御
  const handleInputChange = (fieldApi: any, event: Event) => {
    const target = event.target as HTMLInputElement
    const inputValue = target.value

    // カーソル位置を保存
    const cursorPosition = target.selectionStart || 0

    // 前の値を取得（fieldApiの現在の値）
    const previousValue = fieldApi.state.value || ''

    // 数字以外の文字が含まれているかチェック
    const hasInvalidChars = inputValue.replace(/[\d\s]/g, '').length > 0

    // 無効な文字が含まれている場合は処理を停止し、エラーのみ表示
    if (hasInvalidChars) {
      // バリデーションエラーを表示
      const error = validateCardNumber(inputValue, 'change', null)

      fieldApi.setMeta((prev: any) => ({
        ...prev,
        errorMap: {
          ...prev.errorMap,
          onChange: error,
          onBlur: error,
        },
      }))

      // 入力値は更新せず、前の値を維持
      return
    }

    // 有効な入力の場合のみフォーマット処理
    const formatResult = formatCardNumber(previousValue, inputValue)
    const formattedValue = formatResult.formatted
    const detectedBrand = formatResult.brand

    // 値を更新（フォーマット済みの値）
    fieldApi.handleChange(formattedValue)

    // changeコンテキストでバリデーション実行
    const error = validateCardNumber(formattedValue, 'change', detectedBrand)

    // エラー状態を直接設定
    fieldApi.setMeta((prev: any) => ({
      ...prev,
      errorMap: {
        ...prev.errorMap,
        onChange: error,
        onBlur: error, // onBlurエラーもクリア
      },
    }))

    // カーソル位置を調整（有効な入力の場合のみ）
    setTimeout(() => {
      // 入力が末尾での入力（追加）の場合、カーソルを末尾に移動
      if (cursorPosition === inputValue.length) {
        target.setSelectionRange(formattedValue.length, formattedValue.length)
        return
      }

      // カーソル位置より前の数字の数を数える
      const numbersBeforeCursor = inputValue
        .slice(0, cursorPosition)
        .replace(/\D/g, '').length

      // フォーマット後の文字列でその数字の位置を見つける
      let newCursorPosition = 0
      let numberCount = 0

      for (let i = 0; i < formattedValue.length; i++) {
        if (/\d/.test(formattedValue[i])) {
          numberCount++
          if (numberCount === numbersBeforeCursor) {
            newCursorPosition = i + 1
            break
          }
        }
      }

      // 計算されたポジションが範囲外の場合は末尾に設定
      if (newCursorPosition === 0 && numbersBeforeCursor > 0) {
        newCursorPosition = formattedValue.length
      }

      target.setSelectionRange(newCursorPosition, newCursorPosition)
    }, 0)
  }

  const handleInputBlur = (fieldApi: any, event: Event) => {
    const target = event.target as HTMLInputElement
    const value = target.value

    // ブランド判定
    const numbersOnly = value.replace(/\D/g, '')
    const detectedBrand = detectCardBrand(numbersOnly)

    // blur時にもフォーマットを実行
    const previousValue = fieldApi.state.value || ''
    const formatResult = formatCardNumber(previousValue, value)
    const formattedValue = formatResult.formatted

    // フォーマット済みの値で更新
    if (formattedValue !== value) {
      fieldApi.state.value = formattedValue
      target.value = formattedValue
    }

    // blurコンテキストでバリデーション実行
    const error = validateCardNumber(formattedValue, 'blur', detectedBrand)

    // エラー状態を直接設定
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

<form.Field name="cardNumber">
  {#snippet children(field: any)}
    <div class="mb-4">
      <div class="flex justify-between items-center mb-2">
        <label for="cardNumber" class="text-sm font-medium text-gray-700">
          クレジットカード番号:
        </label>
        {#if detectCardBrand(field.state.value || '')}
          {@const currentBrand = detectCardBrand(field.state.value || '')!}
          <span class="text-sm text-gray-500 font-medium">
            {currentBrand.name}
          </span>
        {/if}
      </div>
      <input
        id="cardNumber"
        name={field.name}
        type="text"
        inputmode="numeric"
        autocomplete="cc-number"
        pattern="[0-9\s]*"
        enterkeyhint="next"
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
      <p class="mt-1 text-xs text-gray-500">例: 1234 5678 9012 3456</p>
      {#if field.state.meta.errors.length > 0}
        <p class="mt-1 text-sm text-red-600" role="alert">
          {field.state.meta.errors[0]}
        </p>
      {/if}
    </div>
  {/snippet}
</form.Field>
