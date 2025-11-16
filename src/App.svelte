<script lang="ts">
  import { createForm } from '@tanstack/svelte-form'
  import CardHolderNameField from './lib/CardHolderNameField.svelte'
  import CreditCardNumberField from './lib/CreditCardNumberField.svelte'
  import ExpiryDateField from './lib/ExpiryDateField.svelte'
  import SecurityCodeField from './lib/SecurityCodeField.svelte'

  // クレジットカードフォーム
  const creditCardForm = createForm(() => ({
    defaultValues: {
      cardNumber: '',
      holderName: '',
      expiryDate: '',
      securityCode: '',
    },
    onSubmit: async ({ value }) => {
      console.log('送信されたクレジットカード情報:', {
        cardNumber: value.cardNumber,
        holderName: value.holderName,
        expiryDate: value.expiryDate,
        securityCode: value.securityCode,
      })
    },
  }))
</script>

<main class="min-h-screen bg-white py-12 px-4">
  <div class="max-w-md mx-auto">
    <h1 class="text-3xl font-bold text-center text-gray-900 mb-8">
      クレカ入力フォームdemo
    </h1>

    <div class="space-y-8">
      <!-- クレジットカード情報入力フォーム -->
      <section>
        <div class="bg-white rounded-lg border border-gray-300 p-6">
          <form
            onsubmit={e => {
              e.preventDefault()
              e.stopPropagation()
              creditCardForm.handleSubmit()
            }}
          >
            <!-- クレジットカード番号 -->
            <CreditCardNumberField form={creditCardForm} />

            <!-- カード名義人 -->
            <CardHolderNameField form={creditCardForm} />

            <!-- 有効期限 -->
            <ExpiryDateField form={creditCardForm} />

            <!-- セキュリティコード -->
            <SecurityCodeField form={creditCardForm} />

            <!-- 送信ボタン -->
            <creditCardForm.Subscribe
              selector={state => [state.canSubmit, state.isSubmitting]}
            >
              {#snippet children([canSubmit, isSubmitting])}
                <button
                  type="submit"
                  disabled={!canSubmit || isSubmitting}
                  class="w-full py-3 px-4 rounded-md border
                         focus:outline-none focus:border-green-500
                         font-medium text-lg
                         {canSubmit && !isSubmitting
                    ? 'bg-green-600 text-white hover:bg-green-700 border-green-600'
                    : 'bg-gray-400 text-gray-200 cursor-not-allowed border-gray-400'}"
                >
                  {isSubmitting ? '処理中...' : '送信(console出力するだけ)'}
                </button>
              {/snippet}
            </creditCardForm.Subscribe>
          </form>
        </div>
      </section>
    </div>
  </div>
</main>

<style>
</style>
