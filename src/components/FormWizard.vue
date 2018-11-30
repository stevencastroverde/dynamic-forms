<template>
  <div>
    <keep-alive>
      <component
        ref="currentStep" 
        :is="currentStep" 
        :wizard-data="form" 
        @update="proccessStep">
      </component>
    </keep-alive>
    <div class="progress-bar">
      <div :style="`width: ${progress}%;`"></div>
    </div>

    <!-- Actions -->
    <div class="buttons">
      <button
        @click="goBack"
        v-if="currentStepNumber > 1"
        class="btn-outlined"
      >Back
      </button>
      <button
        @click="goNext"
        :disabled="!canGoNext"
        class="btn"
      >Next</button>
    </div>

    <pre><code>{{form}}</code></pre>
  </div>
</template>

<script>
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      currentStepNumber: 1,
      canGoNext: false,
      steps: [
        FormPlanPicker,
        FormUserDetails,
        FormAddress,
         FormReviewOrder
      ],
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      }
    }
  },
  computed: {
    progress () {
      return this.currentStepNumber/this.length * 100
    },
    length() {
      return this.steps.length;
    },
    currentStep() {
      return this.steps[this.currentStepNumber - 1];
    },
    isLastStep() {
      return this.currentStepNumber === this.length;
    },
    wizardInProgress() {
      return this.currentStepNumber <= this.length;
    }
  },
  methods: {
    goBack () {
      this.currentStepNumber--;
      this.canGoNext = true;
    },
    goNext () {
      this.currentStepNumber++;
      this.$nextTick(() => {
        this.canGoNext = this.$refs.currentStep.$v.$invalid
      })
      
    },
    proccessStep(stepData) {
      Object.assign(this.form, stepData)
      this.canGoNext = stepData.valid;
    }
  }
}
</script>
