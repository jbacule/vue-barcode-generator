<template>
  <div id="app">
    <Navbar />
    <b-container class="main-content">
      <b-alert v-if="alert" :show="true" :variant="alert.type">{{alert.message}}</b-alert>
      <b-row>
        <b-col sm="5">
          <b-card header="Input Info" >
            <b-form-group size="sm" label="Barcode Type:" label-cols-sm="3">
              <b-form-select v-model="form.barcodeType" :options="barcodeTypesOptions"></b-form-select>
            </b-form-group>
            
            <b-form-group label="Barcode Text:" label-cols-sm="3">
              <b-form-textarea v-model="form.barcodeText" placeholder="Separated by space, comma or next line..." rows="3" max-rows="3" ></b-form-textarea>
            </b-form-group>

            <b-form-group label-cols-sm="3">
              <b-form-checkbox v-model="form.includeText" :value="true" :unchecked-value="false" >Include Text</b-form-checkbox>
            </b-form-group>

            <b-form-group label="Scale (X,Y):" label-cols-sm="3">
              <b-row class="ml-1">
                <b-form-input class="mr-2" style="width: 80px;" type="number" v-model="form.scale"></b-form-input>
                <b-form-input style="width: 80px;" type="number" v-model="form.height"></b-form-input>
              </b-row>
            </b-form-group>

            <b-form-group label="Image Rotation:" label-cols-sm="3">
              <b-form-radio-group id="radio-group-1" v-model="form.imageRotation" :options="imageRotationOption" name="image-rotation-options" ></b-form-radio-group>
            </b-form-group>

            <b-form-group>
              <b-button variant="primary" block @click="onGenerate">Generate</b-button>
              <b-button variant="danger" block @click="onReset">Reset</b-button>
            </b-form-group>
          </b-card>
        </b-col>
        <b-col sm="7"> 
          <b-card header="Result">
            <p v-if="barcodeResults.length < 1">No Barcode Result</p>
            <div v-else class="result">
              <ul class="text-decoration-none">
                <li v-for="(img, index) in barcodeResults" :key="index">
                  <img class="mb-5 mt-2" :src="img" :alt="`result-${index}`">
                </li>
              </ul>
            </div>
          </b-card>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import Navbar from './components/Navbar'
import barcodeTypes from './barcodeTypes'
import bwipjs from 'bwip-js';
import { createCanvas } from 'canvas';
export default {
  name: 'App',
  components: {
    Navbar
  },
  data: () => ({
    barcodeTypesOptions: barcodeTypes,
    imageRotationOption: [
      { value: 'N', text: 'Normal' },
      { value: 'R', text: 'Right' },
      { value: 'L', text: 'Left' },
      { value: 'I', text: 'Inverted' }
    ],
    form: {
      barcodeType: 'auspost',
      barcodeText: '',
      includeText: true,
      scale: 3,
      height: 10,
      imageRotation: 'N'
    },
    barcodeResults: [],
    alert: null
  }),
  methods: {
    onReset(){
      this.barcodeResults = []
      this.form = {
        barcodeType: 'auspost',
        barcodeText: '',
        includeText: true,
        scale: 3,
        height: 10,
        imageRotation: 'N'
      }
    },
    async onGenerate(){
      this.alert = null
      this.barcodeResults = []
      try{
        let barcodeTexts = this.form.barcodeText.split(/[\n\s,]+/);

        for(let x in barcodeTexts){
          let text = barcodeTexts[x];
          let result = await this.generateBarcodeData({...this.form, barcodeText: text})
          this.barcodeResults.push(result)
        }
      }catch(e){
        this.alert = { type: "danger", message: e }
      }
    },
    generateBarcodeData(option){
      const dimension = { width: 300, height: 100 }
      const canvas = createCanvas(dimension.width, dimension.height)
      bwipjs.toCanvas(canvas, {
        bcid: option.barcodeType,
        text: option.barcodeText,
        scale: option.scale,
        height: option.height,
        includetext: option.includeText,
        rotate: option.imageRotation
      });
      return canvas.toDataURL('image/png')
    }
  }
}
</script>

<style>
.main-content{
  margin-top: 4.5em;
}
.col-form-label{
  margin-top: -8px;
}
div.result{
  overflow: auto;
  max-height: 80vh;
}
ul{
  list-style-type: none;
}
</style>