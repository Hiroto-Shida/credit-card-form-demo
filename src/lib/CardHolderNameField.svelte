<script lang="ts">
  // Svelte 5のprops構文を使用
  interface Props {
    form: any
  }

  const { form }: Props = $props()

  // バリデーション関数（入力中用）
  const validateHolderNameOnChange = ({ value }: { value: string }) => {
    if (!value) {
      return 'カード名義人は必須です'
    }
    if (!/^[a-zA-Z\s]+$/.test(value)) {
      return 'アルファベットのみで入力してください'
    }
    // 入力中は姓名チェックをしない
    return undefined
  }

  // バリデーション関数（フォーカスアウト時用）
  const validateHolderNameOnBlur = ({ value }: { value: string }) => {
    if (!value) {
      return 'カード名義人は必須です'
    }
    if (!/^[a-zA-Z\s]+$/.test(value)) {
      return 'アルファベットのみで入力してください'
    }
    if (value.trim().split(' ').length < 2) {
      return '姓名を両方入力してください'
    }
    return undefined
  }
</script>

<form.Field
  name="holderName"
  validators={{
    onChange: validateHolderNameOnChange,
    onBlur: validateHolderNameOnBlur,
  }}
>
  {#snippet children(field: any)}
    <div class="mb-4">
      <label
        for="cardHolderName"
        class="block text-sm font-medium text-gray-700 mb-2"
      >
        カード名義人:
      </label>
      <input
        id="cardHolderName"
        name={field.name}
        type="text"
        inputmode="text"
        autocomplete="cc-name"
        spellcheck="false"
        value={field.state.value}
        onblur={field.handleBlur}
        oninput={e =>
          field.handleChange((e.target as HTMLInputElement)?.value || '')}
        required
        class="w-full px-3 py-2 border rounded-md
               focus:outline-none focus:border-blue-500
               uppercase
               {field.state.meta.errors.length > 0
          ? 'border-red-500'
          : 'border-gray-300'}"
      />
      <p class="mt-1 text-xs text-gray-500">例: TARO YAMADA</p>
      {#if field.state.meta.errors.length > 0}
        <p class="mt-1 text-sm text-red-600" role="alert">
          {field.state.meta.errors[0]}
        </p>
      {/if}
    </div>
  {/snippet}
</form.Field>
