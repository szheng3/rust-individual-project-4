<script setup>
import {computed, ref, watch} from 'vue';
import axios from "axios";
import {ContentLoader} from 'vue-content-loader'
import {notify} from "@kyvg/vue3-notification";


const myForm = ref(null)

const textInput = ref('');
const loading = ref(false);
const showResult = ref(false);
const sliderValue = ref(38);
const isGPU = ref(true);
const result = ref({status: 'success', message: ''});

const models = ref([{name: "T5", token: 500}, {name: "Bart", token: 1000}, {
  name: "LongT5",
  token: 15000
}]);
const defaultSelected = ref("Bart")
const selectedToken = computed(() => {
  return models.value.find((model) => model.name === defaultSelected.value).token
})
const changeSelect = async (a) => {
  if (!!textInput.value) {
    const isValid = await myForm.value.validate();
  }

}

watch(defaultSelected, async (newVal, oldVal) => {
  if (!!textInput.value) {
    const isValid = await myForm.value.validate();
  }
});
const requiredRule = (value) => {


  if (!value) {
    return 'This field is required';
  } else {
    // if (value.split(' ').length > 16384) {
    //   return 'Text length should be less than 16,384 words';
    // }
    if (value.split(' ').length > selectedToken.value) {
      return `Text length should be less than ${selectedToken.value} words`;
    }
    return true;
  }
};


const submitForm = async () => {
  const isValid = await myForm.value.validate();
  if (isValid.valid) {
    // Form is valid, submit it
    loading.value = true;
    try {


      const response = await axios.post('/api/summary', {
        context: textInput.value,
        minlength: Math.round(sliderValue.value / 100 * textInput.value.split(' ').length),
        model: defaultSelected.value,
        is_gpu: isGPU.value
      });
      result.value = await response.data;
      showResult.value = true;


    } catch (error) {

      notify({
        type: "error",
        text: "Ops! something went wrong! Please reduce your text length and try again later.",
      });
    } finally {
      loading.value = false;
    }

  }


};
</script>
<template>

  <v-container>
    <v-row>
      <v-col cols="12"
             sm="6">
        <v-sheet rounded >

          <v-form ref="myForm" @submit.prevent="submitForm" class="pa-3">
            <v-select
                label="Select Models"
                @change="changeSelect($event)"
                v-model="defaultSelected"
                :items="models.map((model) => model.name)"
            ></v-select>
            <v-switch v-model="isGPU" label="GPU" inset></v-switch>

            <v-textarea
                :rules="[requiredRule]"
                rounded
                v-model="textInput"
                label="Enter text to summarize (English)"
                rows="17"
                :loading="loading"></v-textarea>

            <div class="text-caption">
              Longer text will take longer to process. Output words about:
              <strong>{{ Math.round(sliderValue / 100 * textInput.split(' ').length) }}</strong>
            </div>
            <v-slider v-model="sliderValue" min="0" max="100"
                      :thumb-size="15"

                      thumb-label=true></v-slider>


            <v-btn :loading="loading" type="submit" color="primary">Submit</v-btn>
          </v-form>
        </v-sheet>

      </v-col>

      <v-col cols="12"
             sm="6"
      >


        <v-card v-if="!(showResult )" variant="flat">

          <v-card-text>
            <ContentLoader viewBox="0 0 400 350">
              <rect x="0" y="13" rx="4" ry="4" width="400" height="9"/>
              <rect x="0" y="29" rx="4" ry="4" width="100" height="8"/>
              <rect x="0" y="50" rx="4" ry="4" width="400" height="10"/>
              <rect x="0" y="65" rx="4" ry="4" width="400" height="10"/>
              <rect x="0" y="79" rx="4" ry="4" width="100" height="10"/>
              <rect x="0" y="95" rx="4" ry="4" width="400" height="10"/>
              <rect x="0" y="110" rx="4" ry="4" width="400" height="10"/>
              <rect x="0" y="125" rx="4" ry="4" width="100" height="10"/>
              <rect x="0" y="140" rx="5" ry="5" width="400" height="200"/>

            </ContentLoader>
          </v-card-text>
        </v-card>
        <v-card v-if="showResult " variant="flat">
          <v-card-title class="headline">Summarization</v-card-title>
          <v-card-text>{{ result.message }}</v-card-text>
        </v-card>

      </v-col>
    </v-row>


  </v-container>

</template>


