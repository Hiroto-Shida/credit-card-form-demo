<script lang="ts">
  // Svelte 5のprops構文を使用
  interface Props {
    form: any
  }

  const { form }: Props = $props()

  // バリデーション関数（入力中用）
  const validateSecurityCodeOnChange = ({ value }: { value: string }) => {
    if (!value) {
      return undefined // 入力中は必須エラーを出さない
    }
    if (!/^\d+$/.test(value)) {
      return '数字のみで入力してください'
    }
    // 5桁を超えた場合のみエラー表示
    if (value.length > 4) {
      return '4桁以下で入力してください'
    }
    return undefined
  }

  // バリデーション関数（フォーカスアウト時用）
  const validateSecurityCodeOnBlur = ({ value }: { value: string }) => {
    if (!value) {
      return 'セキュリティコードは必須です'
    }
    if (!/^\d+$/.test(value)) {
      return '数字のみで入力してください'
    }
    if (value.length < 3 || value.length > 4) {
      return '3桁または4桁で入力してください'
    }
    return undefined
  }
</script>

<form.Field
  name="securityCode"
  validators={{
    onChange: validateSecurityCodeOnChange,
    onBlur: validateSecurityCodeOnBlur,
  }}
>
  {#snippet children(field: any)}
    <div class="mb-4">
      <label
        for="securityCode"
        class="block text-sm font-medium text-gray-700 mb-2"
      >
        セキュリティコード (CVV):
      </label>
      <input
        id="securityCode"
        name={field.name}
        type="password"
        inputmode="numeric"
        autocomplete="cc-csc"
        pattern="[0-9]*"
        value={field.state.value}
        onblur={field.handleBlur}
        oninput={e =>
          field.handleChange((e.target as HTMLInputElement)?.value || '')}
        required
        class="w-full px-3 py-2 border rounded-md
               focus:outline-none focus:border-blue-500
               font-mono
               {field.state.meta.errors.length > 0
          ? 'border-red-500'
          : 'border-gray-300'}"
      />
      <p class="mt-1 text-xs text-gray-500">3桁または4桁の数字</p>
      {#if field.state.meta.errors.length > 0}
        <p class="mt-1 text-sm text-red-600" role="alert">
          {field.state.meta.errors[0]}
        </p>
      {/if}
    </div>
  {/snippet}
</form.Field>
