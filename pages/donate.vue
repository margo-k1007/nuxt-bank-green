<template>
  <div class="page">
    <div class="page-fade-in overflow-hidden max-w-screen">
      <div class="bg-gradient-to-b from-sushi-50 to-sushi-100 pb-28 pt-36">
        <div
          class="contain grid grid-cols-1 md:grid-cols-2 justify-center items-center pb-4 lg:pb-0 mb-4 gap-16"
        >
          <div class="space-y-8">
            <PrismicImage
              v-if="donation?.data.photo"
              class="w-full h-full object-contain object-top"
              alt="donation-image"
              :field="donation?.data.photo"
            />
            <PrismicText
              :field="donation?.data.title"
              wrapper="h2"
              class="font-semibold text-xl md:text-3xl tracking-wider"
              fallback="Help us build a greener future!"
            />
            <PrismicRichText
              v-if="donation?.data.description"
              :field="donation?.data.description"
              class="prose sm:prose-lg xl:prose-xl prose-blurb"
            />
            <div v-else class="prose sm:prose-lg xl:prose-xl prose-blurb">
              <p>
                By supporting Bank.Green’s mission, you'll empower individuals and businesses to make
                responsible financial decisions, channeling their deposits towards green financial institutions.
                Bank.Green is a project of a registered charity and all U.S. donations are tax-deductible.
              </p>
              <p>
                Your donation will help us raise awareness, provide resources, and foster a global
                community dedicated to protecting our planet.
              </p>
              <p>
                <em>Bank.Green is a project of a registered charity and all U.S. donations are tax-deductible.</em>
              </p>
            </div>
          </div>
          <div
            class="relative w-full flex items-center justify-center bg-leaf-700 rounded-2xl px-6 lg:px-10 py-8 text-gray-50 text-center"
          >
            <div class="absolute -top-4 md:-top-8 -right-4 md:-right-8">
              <img
                class="h-12 md:h-20 w-auto"
                src="/img/logos/bankgreen-logo.png"
                alt="Bank Green"
              >
            </div>
            <form
              class="flex flex-col rounded-xl"
              @submit.prevent.stop="submit"
            >
              <PrismicText
                :field="donation?.data.donation_title"
                wrapper="h1"
                class="font-semibold text-xl md:text-3xl tracking-wider text-white"
                fallback="Donate to Bank.Green"
              />
              <PrismicRichText
                :field="donation?.data.donation_description"
                class="prose sm:prose-lg xl:prose-xl prose-blurb text-white mt-4 text-justify"
                fallback="...and make a big difference in the world. Your donation will give us greater
                capacity to green the banking sector and protect our collective future."
              />
              <div class="grid grid-cols-2 md:grid-cols-2 px-4 gap-4 mt-8">
                <div
                  v-for="_option in methodOptions"
                  :key="_option.value"
                  class="inline-block"
                >
                  <input
                    :id="_option.value.toString()"
                    v-model="selectedMethod"
                    type="radio"
                    :value="_option.value"
                    class="hidden"
                  >
                  <label
                    class="md:w-auto flex justify-center block font-medium focus:outline-none border border-transparent focus:ring-2 focus:ring-offset-2 focus:ring-leaf-500 p-2 md:p-4 text-center w-full rounded-lg shadow-green capitalize cursor-pointer"
                    :class="
                      selectedMethod !== _option.value
                        ? 'bg-leaf-500 hover:bg-white  text-white hover:text-leaf-500'
                        : 'bg-white text-leaf-500'
                    "
                    :for="_option.value.toString()"
                  >
                    {{ _option.label }}
                  </label>
                </div>
                <div
                  v-for="_option in donationOptions"
                  :key="_option.value"
                  class="inline-block"
                >
                  <input
                    :id="_option.value.toString()"
                    v-model="selectedAmount"
                    type="radio"
                    :value="_option.value"
                    class="hidden"
                  >
                  <label
                    class="md:w-auto flex justify-center block font-medium focus:outline-none border border-transparent focus:ring-2 focus:ring-offset-2 focus:ring-leaf-500 p-2 md:p-4 text-center w-full rounded-lg shadow-green capitalize cursor-pointer"
                    :class="
                      selectedAmount !== _option.value
                        ? 'bg-leaf-500 hover:bg-white  text-white hover:text-leaf-500'
                        : 'bg-white text-leaf-500'
                    "
                    :for="_option.value.toString()"
                  >
                    {{ _option.label }}
                  </label>
                </div>
              </div>
              <div class="flex flex-col gap-4 mt-12 px-4" :class="!isStripeLoaded && 'hidden'">
                <TextField
                  v-model="email"
                  name="email"
                  type="email"
                  :placeholder="'youremail@address.com'"
                  :warning="warningsMap.email"
                  :dark="true"
                  :required="false"
                />
                <CheckboxSection
                  v-model="isAgreeMarketing"
                  class="col-span-2"
                  name="isAgreeMarketing"
                  :warning="warningsMap.isAgreeMarketing"
                  :dark="true"
                >
                  I wish to receive more information via email from
                  Bank.Green.
                </CheckboxSection>
                <div
                  id="stripe-payment-element"
                  class="bg-gray-50 px-6 py-8 rounded-xl"
                />
              </div>
              <button
                type="submit"
                class="button-green w-full md:w-auto mt-12 flex justify-center "
              >
                <PrismicText
                  wrapper="span"
                  class="text-2xl font-semibold text-primary-dark"
                  :field="donation?.data.donation_button"
                  fallback="Donate Now"
                />
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import TextField from '@/components/forms/TextField.vue'
import CheckboxSection from '@/components/forms/CheckboxSection.vue'

interface DonationOption<T> {
  label: string;
  value: T;
}

const methodOptions: DonationOption<string>[] = [
  { label: 'One-time Donation', value: 'one-time' },
  { label: 'Recurring Donation', value: 'recurring' }
]

const donationOptions: DonationOption<number>[] = [
  { label: '$25', value: 25 },
  { label: '$50', value: 50 },
  { label: '$100', value: 100 },
  { label: '$200', value: 200 },
  { label: '$500', value: 500 },
  { label: '$1000', value: 1000 }
]

const selectedMethod = ref<string>('one-time')
const selectedAmount = ref<number | null>(null)
const {
  email,
  warningsMap,
  isAgreeMarketing,
  send
} = useContactForm('', ['email'], ref({}))

const isOneTimePayment = computed(
  () => selectedMethod.value === 'one-time' && selectedAmount.value != null
)

const isRecurring = computed(
  () => selectedMethod.value === 'recurring' && selectedAmount.value != null
)

const { client } = usePrismic()
const { data: donation } = await useAsyncData('donation', () =>
  client.getSingle('donationpage')
)
console.log('data', donation.value?.data)
usePrismicSEO(donation.value?.data)

const stripePublishableKey = useRuntimeConfig().public.STRIPE_PUBLISHABLE_KEY

const { isStripeLoaded, initOneTimePayment, handleSubmit } = useStripe(
  stripePublishableKey,
  'stripe-payment-element'
)

watch(
  () => selectedAmount.value,
  (newVal, _) => {
    if (isOneTimePayment.value && newVal !== null) {
      isStripeLoaded.value = false
      initOneTimePayment(newVal)
    }
  }
)

watch(
  () => selectedMethod.value,
  (newVal, oldVal) => {
    const _isOneTimePayment = newVal !== oldVal && newVal === 'one-time'
    const _isRecurringPayment = newVal !== oldVal && newVal === 'recurring'

    if (_isRecurringPayment && isStripeLoaded.value) { isStripeLoaded.value = false }

    if (
      _isOneTimePayment &&
      selectedAmount.value != null &&
      isStripeLoaded.value
    ) {
      isStripeLoaded.value = false
      initOneTimePayment(selectedAmount.value)
    }
  }
)

const handleOneTimePayment = async () => {
  if (selectedAmount.value == null) { /* empty */ } else if (isStripeLoaded.value) { handleSubmit(isAgreeMarketing.value, email.value) } else {
    await initOneTimePayment(selectedAmount.value)
  }
}

const handleRecurringPayment = async () => {
  const response = await $fetch('/api/create-checkout-session', {
    method: 'POST',
    body: {
      amount: selectedAmount.value
    }
  })
  if (response.redirectURL) { window.location.href = response.redirectURL }
}

const submit = () => {
  if (isOneTimePayment.value) {
    if (email.value.length > 0) { send() }
    handleOneTimePayment()
  } else if (isRecurring.value) {
    handleRecurringPayment()
  }
}
</script>
