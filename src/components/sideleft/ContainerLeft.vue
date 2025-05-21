<script setup lang="ts">
import { ref, watch } from 'vue'
import { gsap } from 'gsap'

type LoanType = 'repayment' | 'interest-only'

interface CalculationResult {
  monthly: number
  total: number
}

const INITIAL_TERM = null
const INITIAL_RATE = null
const INITIAL_TYPE: LoanType = 'repayment'

const amountInput = ref('')
const amount = ref<number | null>(null)
const term = ref<number | null>(INITIAL_TERM)
const rate = ref<number | null>(INITIAL_RATE)
const type = ref<LoanType>(INITIAL_TYPE)
const showAmountError = ref(false)

const sanitizeInput = (input: string) => input.replace(/\s/g, '')
const hasInvalidCharacters = (input: string) => /[^0-9.,]/.test(input)
const shouldShowError = () => window.innerWidth > 1010

const parseCurrency = (input: string): number | null => {
  const cleanDecimal = input.replace(/,/g, '').replace(/\./g, '')
  const [integer, decimal] = input.split('.')

  return decimal ? parseFloat(`${integer.replace(/\D/g, '')}.${decimal}`) : parseFloat(cleanDecimal)
}

watch(amountInput, (newValue) => {
  const cleanedValue = sanitizeInput(newValue)

  if (hasInvalidCharacters(cleanedValue)) {
    showAmountError.value = shouldShowError()
    amount.value = null
    return
  }

  showAmountError.value = false
  amount.value = parseCurrency(cleanedValue)
})

const calculateRepayment = (principal: number, monthlyRate: number, months: number) => {
  const discountFactor = Math.pow(1 + monthlyRate, -months)
  return (principal * monthlyRate) / (1 - discountFactor)
}

const calculateInterestOnly = (principal: number, monthlyRate: number) => principal * monthlyRate

const emit = defineEmits<{
  (e: 'calculated', result: CalculationResult): void
  (e: 'clear'): void
}>()

function calculate() {
  if (!amount.value || !term.value || !rate.value) return

  const principal = amount.value
  const months = term.value * 12
  const monthlyRate = rate.value / 100 / 12

  const monthlyPayment =
    type.value === 'repayment'
      ? calculateRepayment(principal, monthlyRate, months)
      : calculateInterestOnly(principal, monthlyRate)

  emit('calculated', {
    monthly: monthlyPayment,
    total: monthlyPayment * months,
  })
}

function clearAll() {
  amountInput.value = ''
  amount.value = null
  term.value = INITIAL_TERM
  rate.value = INITIAL_RATE
  type.value = INITIAL_TYPE

  const active = document.activeElement
  if (active && 'blur' in active && typeof (active as HTMLElement).blur === 'function') {
    (active as HTMLElement).blur()
  }

  emit('clear')
}
</script>

<template>
  <div>
    <div v-if="showAmountError" class="amount-error-notification">
      ⚠️ Please enter only numbers in the amount field.
    </div>

    <div class="mortgage_cl-header">
      <a href="./index.html">
        <h1>Mortgage Calculator</h1>
      </a>
      <span @click="clearAll" style="cursor: pointer">Clear all</span>
    </div>

    <form class="mortgage_cl-form" @submit.prevent="calculate">
      <div class="mortgage_cl-form-amount">
        <label>Mortage Amount</label>
        <div class="mortgage_cl-form-amount--input">
          <p>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon icon-tabler icons-tabler-outline icon-tabler-currency-euro"
            >
              <path stroke="none" d="M0 0h24v24H0z" fill="none" />
              <path d="M17.2 7a6 7 0 1 0 0 10" />
              <path d="M13 10h-8m0 4h8" />
            </svg>
          </p>
          <input type="text" id="mgg-input-amount" required v-model="amountInput" step="any" />
        </div>
      </div>

      <div class="mortgage_cl-form-term-and-rates">
        <div class="mortgage_cl-form-term">
          <label>Mortgage Term</label>
          <div class="term-input-ex">
            <input type="number" required v-model="term" />
            <span style="color: #000;">years</span>
          </div>
        </div>

        <div class="mortgage_cl-form-rate">
          <label>Interest Rate</label>
          <div class="term-input-ex">
            <input type="number" required v-model="rate" step="any" />
            <p>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="#000"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                class="icon icon-tabler icons-tabler-outline icon-tabler-percentage"
              >
                <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                <path d="M17 17m-1 0a1 1 0 1 0 2 0a1 1 0 1 0 -2 0" />
                <path d="M7 7m-1 0a1 1 0 1 0 2 0a1 1 0 1 0 -2 0" />
                <path d="M6 18l12 -12" />
              </svg>
            </p>
          </div>
        </div>
      </div>

      <div class="mortgage_cl_form-types">
        <label>Mortgage Type</label>
        <div class="mortgage_cl_form-inputs">
          <div class="type-repayment">
            <input type="radio" name="mortgageType" value="repayment" v-model="type" />
            <span>Repayment</span>
          </div>
          <div class="type-interest-only">
            <input type="radio" name="mortgageType" value="interest-only" v-model="type" />
            <span>Interest Only</span>
          </div>
        </div>
      </div>

      <button class="mortgage_cl_form-calculate-btn">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="#000"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          class="icon icon-tabler icons-tabler-outline icon-tabler-calculator"
        >
          <path stroke="none" d="M0 0h24v24H0z" fill="none" />
          <path
            d="M4 3m0 2a2 2 0 0 1 2 -2h12a2 2 0 0 1 2 2v14a2 2 0 0 1 -2 2h-12a2 2 0 0 1 -2 -2z"
          />
          <path d="M8 7m0 1a1 1 0 0 1 1 -1h6a1 1 0 0 1 1 1v1a1 1 0 0 1 -1 1h-6a1 1 0 0 1 -1 -1z" />
          <path d="M8 14l0 .01" />
          <path d="M12 14l0 .01" />
          <path d="M16 14l0 .01" />
          <path d="M8 17l0 .01" />
          <path d="M12 17l0 .01" />
          <path d="M16 17l0 .01" />
        </svg>
        Calculate Repayments
      </button>
    </form>
  </div>
</template>

<style scoped>
label {
  color: rgb(27, 29, 37);
  font-weight: 600;
}

.amount-error-notification {
  position: fixed;
  top: 0px;
  right: 0;
  background: #fffbe7;
  color: #b89b0f;
  border: 1px solid #dbdb2d;
  border-radius: 8px;
  padding: 12px 20px;
  font-size: 16px;
  font-weight: 600;
  z-index: 9999;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  display: none;
}
@media (min-width: 1011px) {
  .amount-error-notification {
    display: block;
  }
}

.mortgage_cl-header {
  display: flex;
  justify-content: space-between;
  align-items: center;

  & > a,
  h1 {
    color: hsl(244, 23%, 12%);
    text-decoration: none;
    font-weight: 700;
  }

  & > span {
    text-decoration: underline;
  }
}

.mortgage_cl-form {
  margin-top: 26px;

  & > .mortgage_cl-form-amount {
    display: flex;
    gap: 6pt;
    flex-direction: column;

    & > .mortgage_cl-form-amount--input {
      display: flex;
      align-items: center;
      border: 1px solid #a7b5b8;
      border-radius: 6px;

      & > p {
        border-radius: 5px 0px 0px 5px;
        text-align: center;
        width: 45px;
        padding: 6px;
        background-color: transparent;
        transition: background 0.2s;
      }

      & > #mgg-input-amount {
        background: #fff;
        outline: none;
        border: none;
        padding: 12px;
        width: 100%;
        font-size: 16px;
        border-radius: 0 5px 5px 0px;
      }
    }
  }
}

.mortgage_cl-form-term-and-rates {
  margin-top: 16px;
  display: flex;
  justify-content: space-between;

  & > .mortgage_cl-form-term {
    display: flex;
    gap: 6pt;
    flex-direction: column;

    & > .term-input-ex {
      display: flex;
      align-items: center;
      border: 1px solid hsl(191, 11%, 69%);
      border-radius: 6px;

      & > input {
        outline: none;
        border: none;
        padding: 10px;
        width: 150px;
        font-size: 16px;
        border-radius: 5px 0px 0px 5px;
      }

      & > span {
        border-radius: 0px 5px 5px 0px;
        text-align: center;
        width: 45px;
        padding: 6px;
        font-weight: 600;
        background-color: hsl(200, 87%, 94%);
      }
    }
  }

  & > .mortgage_cl-form-rate {
    display: flex;
    gap: 6pt;
    flex-direction: column;

    & > .term-input-ex {
      display: flex;
      align-items: center;
      border: 1px solid hsl(191, 11%, 69%);
      border-radius: 6px;

      & > input {
        outline: none;
        border: none;
        padding: 8px;
        width: 150px;
        font-size: 16px;
        border-radius: 5px 0px 0px 5px;
      }

      & > p {
        border-radius: 0px 5px 5px 0px;
        text-align: center;
        width: 45px;
        padding: 4px;
        background-color: hsl(200, 87%, 94%);

        & > svg {
          transform: translateY(5px);
        }
      }
    }
  }
}

.mortgage_cl_form-types {
  margin-top: 10px;
  display: flex;
  gap: 6pt;
  flex-direction: column;

  & > .mortgage_cl_form-inputs {
    display: flex;
    gap: 6pt;
    flex-direction: column;
    & > .type-repayment,
    .type-interest-only {
      display: flex;
      gap: 6pt;
      align-items: center;
      border: 1px solid hsl(191, 11%, 69%);
      border-radius: 6px;
      padding: 10px;
      transition:
        border 0.2s,
        background 0.2s;
    }
  }
}

.mortgage_cl_form-calculate-btn {
  background-color: #dbdb2d;
  border: none;
  cursor: pointer;
  color: #293b0f;
  margin-top: 16px;
  width: 100%;
  display: flex;
  gap: 6px;
  align-items: center;
  font-size: 16px;
  font-weight: 600;
  padding: 12px;
  border-radius: 60px;
  justify-content: center;
}

.mortgage_cl-form-amount--input:focus-within {
  border: 2px solid #dbdb2d;
  background-color: #c9db26;
}

.mortgage_cl-form-amount--input:focus-within > p {
  background-color: #f9fbe7;
}

.term-input-ex:focus-within {
  border: 1px solid hsl(191, 11%, 69%) !important;
  background: transparent !important;
}
.term-input-ex:focus-within > span,
.term-input-ex:focus-within > p {
  background: inherit !important;
}

.type-repayment:has(input[type='radio']:focus),
.type-repayment:has(input[type='radio']:checked),
.type-interest-only:has(input[type='radio']:focus),
.type-interest-only:has(input[type='radio']:checked) {
  border: 2px solid #dbdb2d;
  background-color: #f9fbe7;
}

.type-repayment span,
.type-interest-only span {
  background: none !important;
  border: none !important;
  font-weight: 400;
}

.type-repayment input[type='radio']:checked + span,
.type-interest-only input[type='radio']:checked + span {
  font-weight: 600;
}
</style>
