<template>
  <div class="q-pa-md q-gutter-xl">
    <q-btn label="Confirm something" color="primary" @click="showDialog = true" class="q-ma-lg" />
    
    <q-dialog v-model="showDialog" stack-buttons prevent-close @cancel="onCancel" @ok="onOk" @show="onShow" @hide="onHide">
      <span slot="title">Favorite Superhero</span>
      <!-- This or use "message" prop on <q-dialog> -->
        <span slot="message">What is your superhero of choice?</span>

        <div slot="body">
          <q-field
            icon="account_circle"
            helper="We need your name so we can send you to the movies."
            label="Your name"
            :label-width="3"
            :error="nameError"
          >
            <q-input v-model="name" />
          </q-field>
        </div>

        <template slot="buttons" slot-scope="props">
          <q-btn color="primary" label="Choose Superman" @click="choose(props.ok, 'Superman')" />
          <q-btn color="black" label="Choose Batman" @click="choose(props.ok, 'Batman')" />
          <q-btn color="negative" label="Choose Spiderman" @click="choose(props.ok, 'Spiderman')" />
          <q-btn flat label="No thanks" @click="props.cancel" />
        </template>
    </q-dialog>
    
    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-fab
        color="purple"
        icon="keyboard_arrow_up"
        direction="up"
      >
        <q-fab-action
          color="primary"
          @click="onOk"
          icon="mail"
        />

        <q-fab-action
          color="secondary"
          @click="onOk"
          icon="alarm"
        />
      </q-fab>
    </q-page-sticky>
    
  </div>
</template>

<script>
//import { Dialog } from 'quasar'
//import CustomDialog from 'src/components/custom/CustomDialog'
//import Vue from "vue"
//Vue.component("CustomDialog", CustomDialog)

export default {
  name: 'PageIndex',
  data () {
    return {
      showDialog: false
    }
  },
  methods: {
    onOk () {
      console.log('ok')
    },
    onCancel () {
      console.log('cancel')
    },
    onShow () {
      console.log('show')
    },
    onHide () {
      console.log('hide')
    }
  }
}
</script>
