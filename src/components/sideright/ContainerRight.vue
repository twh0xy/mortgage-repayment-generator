<script setup lang="ts">
import { ref, watch, nextTick, type Ref } from 'vue'
import { gsap } from 'gsap'

type AnimationConfig = {
  from: gsap.TweenVars
  to: gsap.TweenVars
}

interface AnimationParams {
  element: Ref<HTMLElement | null>
  config: AnimationConfig
}

const ENTRANCE_ANIMATION: AnimationConfig = {
  from: { opacity: 0, y: 40 },
  to: { opacity: 1, y: 0, duration: 0.7, ease: 'power2.out' },
}

const EXIT_ANIMATION: AnimationConfig = {
  from: { opacity: 0, y: -40 },
  to: { opacity: 1, y: 0, duration: 0.7, ease: 'power2.out' },
}

const props = defineProps<{
  result?: { monthly: number; total: number } | null
}>()

const resultBox = ref<HTMLElement | null>(null)
const heroBox = ref<HTMLElement | null>(null)

const executeAnimation = async ({ element, config }: AnimationParams) => {
  await nextTick()
  if (element.value) {
    gsap.fromTo(element.value, config.from, config.to)
  }
}

watch(
  () => props.result,
  async (newVal, oldVal) => {
    const isResultAppearing = newVal && !oldVal
    const isResultDisappearing = !newVal && oldVal

    if (isResultAppearing) {
      await executeAnimation({
        element: resultBox,
        config: ENTRANCE_ANIMATION,
      })
    }

    if (isResultDisappearing) {
      await executeAnimation({
        element: heroBox,
        config: EXIT_ANIMATION,
      })
    }
  },
  { flush: 'post' },
)
</script>

<template>
  <div class="mortgage_cl_result-hero" ref="heroBox">
    <template v-if="result">
      <div class="mortgage_cl_result-header">
        <h2>Your results</h2>
        <p class="mortgage_cl_result-desc">
          Your results are shown below based on the information you provided. To adjust the results,
          edit the form and click "calculate repayments" again.
        </p>
      </div>
      <div class="mortgage_cl_result-box" ref="resultBox">
        <div class="mortgage_cl_result-label">Your monthly repayments</div>
        <div class="mortgage_cl_result-value">
          £{{
            result.monthly.toLocaleString(undefined, {
              minimumFractionDigits: 2,
              maximumFractionDigits: 2,
            })
          }}
        </div>
        <hr class="mortgage_cl_result-divider" />
        <div class="mortgage_cl_result-label-small">Total you'll repay over the term</div>
        <div class="mortgage_cl_result-total">
          £{{
            result.total.toLocaleString(undefined, {
              minimumFractionDigits: 2,
              maximumFractionDigits: 2,
            })
          }}
        </div>
      </div>
    </template>
    <template v-else>
      <div>
        <img src="./illustration-empty.svg" alt="" />
      </div>
      <div class="mortgage_cl_result-text">
        <h3>Result shown here</h3>
        <p>
          Complete the form and click "calculate repayments" to see what your monthly repayments
          would be.
        </p>
      </div>
    </template>
  </div>
</template>

<style scoped>
.mortgage_cl_result-hero {
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;

  & > .mortgage_cl_result-header {
    margin-bottom: 24px;

    & > h2 {
      color: #fff;
      font-size: 24px;
      font-weight: 700;
      margin-bottom: 4px;
    }
    & > .mortgage_cl_result-desc {
      color: #b8c7d1;
      font-size: 15px;
      margin: 0;
    }
  }

  & > .mortgage_cl_result-box {
    background: #0e2431;
    border-radius: 10px;
    padding: 32px 28px 24px 28px;
    margin-top: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    border-top: 3px solid #dbdb2d;
    display: flex;
    flex-direction: column;
    gap: 18px;

    & > .mortgage_cl_result-label {
      color: #b8c7d1;
      font-size: 16px;
      font-weight: 500;
    }

    & > .mortgage_cl_result-value {
      color: #dbdb2d;
      font-size: 40px;
      font-weight: 700;
      letter-spacing: -1px;
    }

    & > .mortgage_cl_result-divider {
      border: none;
      border-top: 1px solid #2b4452;
      margin: 10px 0;
    }

    & > .mortgage_cl_result-label-small {
      color: #b8c7d1;
      font-size: 15px;
      margin-bottom: 2px;
    }

    & > .mortgage_cl_result-total {
      color: #fff;
      font-size: 24px;
      font-weight: 700;
    }
  }

  & > .mortgage_cl_result-text {
    width: 380px;
    margin: 0 auto;
    text-align: center;

    & > h3 {
      color: hsl(0, 0%, 100%);
      font-weight: 600;
      font-size: 28px;
    }

    & > p {
      color: hsl(203, 25%, 60%);
      font-weight: 400;
    }
  }

  & > img,
  & img {
    width: 260px;
    display: block;
    margin: 0 auto 24px auto;
  }
}
</style>
