<template>
  <div>
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component
          ref="currentStep" 
          :is="currentStep" 
          :wizard-data="form"
          @updateAsyncState="updateAsyncState" 
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
          @click="nextButtonAction"
          :disabled="!canGoNext"
          class="btn"
        >{{isLastStep ? "Complete Order": "Next"}}</button>
      </div>

      <pre><code>{{form}}</code></pre>
    </div>
    <div v-else>
      <h1 clas="title">
        Thank you!
      </h1>
      <h2 class="subtitle">
        We Look forward to shipping you your first box!
        </h2> 
        <p class="text-center">
          <a href="https://vueschool.io" class="btn" target="_blank">Go somewhere cool!</a>
        </p>
    </div>
    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="">
        <p>Please wait, we're hitting our servers</p>
      </div>
    </div>
  </div>
</template>

<script>
import { postFormToDB } from '../api/index'
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
      asyncState: null,
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
    nextButtonAction() {
      if (this.isLastStep) {
        return this.submitOrder()
      }
      this.goNext();
    },
    proccessStep({valid,data}) {
      Object.assign(this.form, data)
      this.canGoNext = valid;
    },
    submitOrder(){
      this.asyncState = 'pending'
      postFormToDB(this.form)
      .then(() => {
        console.log('form submit')
        this.asyncState = 'success';
      })
      this.currentStepNumber++;
    },
    updateAsyncState (state) {
      this.asyncState = state;
    }
  }
}
</script>
