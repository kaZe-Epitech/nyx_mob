<template>
  <q-page class="bg-grey-5">
    
<!-- SINGLE PURCHASE ORDER -->
    <div v-if="poDisplayed" class="q-pa-xs">
      <!-- CART / COLLAPSIBLE SUPPLIER INFOS -->
      <q-card flat bordered class="bg-white q-ma-md q-pa-md">
        <q-collapsible icon="perm_identity" :label="this.currentPurchaseOrder.supplier">
          <div>
            <div><span class="caption">Date commande :</span> <b>{{ goodLookingDate(this.currentPurchaseOrder.expected_date) }}</b></div>
            <div><span class="caption">Producteur :</span> <b>{{ this.currentPurchaseOrder.supplier }}</b></div>
            <div><span class="caption">Contact : </span> <b>{{ this.currentPurchaseOrder.supplier }}</b></div>
            <div><span class="caption">Numéro de commande :</span> <b>{{ this.currentPurchaseOrder.number }}</b></div>
            <div><span class="caption">Statut actuel :</span> <b>{{ this.currentPurchaseOrder.status }}</b></div>
            <div><span class="caption">Type de bon :</span> <b>{{ this.currentPurchaseOrder.type }}</b></div>
            <hr class="caption">
            <p class="caption text-center">
              <b>Nb d'items dans le panier : {{ this.currentPurchaseOrder.length }}</b>
            </p>
          </div>
        </q-collapsible>      
             
        <!-- CART ITEMS LIST -->
        <div class="">
          <q-list class="bg-white" separator bordered striped>
            <q-item :class="getTheColor(item, 'bg')" v-for="(item, index) in this.currentPurchaseOrder.line_items" :key="item.product_id" clickable v-ripple style="padding: 8px; min-height:60px;">
              <div color="yellow" style="max-width: 150px;">
                <div :style="getTheColor(item, 'name')" name="q-item-label">{{ item.full_title }}</div>
              </div>
              <div class="absolute-right">
                <div class="q-mr-xs text-right" style="display: inline-block; min-width: 31px;">
                  <q-chip small pointing="right" style="width: 45px" :color="getTheColor(item, 'chip')">
                    <b>{{ item.quantity }}</b>
                  </q-chip>
                </div>
                <div v-model="item.received" class="q-mr-xs text-center" style="display: inline-block; min-width: 31px;">
                  {{ item.received }}
                </div>
                <div>
                  <!-- old icon : settings -->
                  <q-btn size="30px" flat dense square icon="priority_high" :color="getTheColor(item, 'btn')" @click="openProblemBox(index)" />
                  <q-btn size="30px" flat dense square icon="done" :color="getTheColor(item, 'btn')" @click="lineIsOk(index)" />
                </div>
              </div>
            </q-item>          
          </q-list>
        </div>
        <!-- BOUTTONS RETOUR & SEND DATA -->
        <div class="q-mt-md text-center">
          <q-btn label="Retour" color="primary" icon="arrow_back_ios" class="q-mx-lg" @click="backToList" />
          <q-btn label="Valider le bon" color="light-blue-2" icon-right="send" class="q-mx-lg" :disable="!hasPoChanged" @click="sendData" />
        </div>
      </q-card>

      <!-- REPORT PROBLEM DIALOG BOX -->
      <q-dialog v-model="problemBox" prevent-close @cancel="onCancelProblem" @ok="onOkProblem">
        <span slot="title">Quantité reçue</span>
        <div slot="body" class="q-py-lg">
          <!-- <span class="on-left">0</span><span class="float-right">{{ currentEditQty }}</span> -->
          <!-- <q-slider v-model="currentEditQty_received" :min="0" :max="currentEditQty" :step="1" label snap markers /> -->
          <div>
            <scroll-picker
            :options="scrollPickerOptions"
            v-model="currentEditQty_received"
            :drag-sensitivity="1"
            :touch-sensitivity="1"
            :scroll-sensitivity="0.5" />
          </div>
          
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel" />
          <q-btn flat label="Valider" color="primary" @click="props.ok"/>
        </template>
      </q-dialog>

    </div>

<!-- PURCHASE ORDERS LIST -->
    <div v-else>

      <!-- REFERENCE DATE -->
      <div class="date-ref bg-grey-8 q-py-sm q-px-md text-white">
          <div v-if="this.dateFromShort === this.dateToShort" class="text-center">
            Date : <b>{{ goodLookingDate(this.dateFrom) }}</b>
          </div>
          <div v-else class="text-center">
            Période : du <b>{{ goodLookingDate(this.dateFrom) }}</b> au <b>{{ goodLookingDate(this.dateTo) }}</b>
          </div>
      </div>
      <!-- INFINITE SCROLL / PO LIST-->
      <q-infinite-scroll :handler="loadMore" v-if="poList.length">
        <div v-for="purchaseOrder in poList" :key="purchaseOrder.id" class="q-py-xs q-px-sm">
          <q-card class="bg-white" style="max-width: 700px;">
            <div v-ripple @click="getPurchaseOrder(purchaseOrder._id, purchaseOrder._index)" class="cursor-pointer relative-position ">
              <q-card-main style="padding: 10px; display: flex;">
                <div class="caption" style="width: 50px; margin-right: 5px;">
                  {{ purchaseOrder._source.expected_date | formatDate }}
                </div>
                <div class="caption" style="width: 210px;">
                  <b>{{ purchaseOrder._source.supplier }}</b>
                </div>
                <div class="caption" style="width: 100px;">
                  {{ checkNumber(purchaseOrder._source.number) }}
                </div>
                <q-chip dense :color="chipColor(purchaseOrder._source.status)" text-color="white" style="width: 90px; max-height: 25px;" class="text-center">
                  <b>{{ chipStatus(purchaseOrder._source.status) }}</b>
                </q-chip>
              </q-card-main>
            </div>
          </q-card>
        </div>
      </q-infinite-scroll>

      <!-- BOX IF DATA RECEIVED-->
      <div v-else class="q-pa-md">
        <q-card class="caption bg-white fixed-center">
          <q-card-main class="q-pa-md">
          <b>Aucun résultat trouvé</b>
          </q-card-main>
        </q-card>

      </div>
    </div>

    <!-- STICKY BUTTON -->
    <q-fab v-if="!poDisplayed" v-model="overlayFab" color="primary" icon="calendar_today" direction="up" class="fixed" style="right: 25px; bottom: 25px; z-index: 10;" @click="toggleFab">
      <!-- TODAY BUTTON -->
      <q-fab-action color="primary" @click="onToday" icon="today" class="my-fab-action">
        Aujourd'hui
      </q-fab-action>
      <!-- DATE BUTTON -->
      <q-fab-action color="primary" @click="pickDateDialog = true" icon="event" class="my-fab-action">
        Date
      </q-fab-action>
      <!-- RANGE DATE BUTTON -->
      <q-fab-action color="primary" @click="pickRangeDateDialog = true" icon="date_range" class="my-fab-action">
        Période
      </q-fab-action>
    </q-fab>

    <!-- DATE PICKER DIALOG BOX -->
    <q-dialog v-model="pickDateDialog" @ok="onOkDatePicker">
      <span slot="title">Choisir un jour</span>
      <div slot="body">
        <q-datetime v-model="curSelDate" format="DD/MM/YYYY" />
      </div>
      <template slot="buttons" slot-scope="props">
        <q-btn flat label="Valider" color="primary" @click="props.ok"/>
      </template>
    </q-dialog>

    <!-- RANGE DATE PICKER DIALOG BOX -->
    <q-dialog v-model="pickRangeDateDialog" @ok="onOkRangeDatePicker">
      <span slot="title">Choisir une période</span>
      <div slot="body">
        <q-datetime v-model="dateFrom" stack-label="Date de début" />
        <q-datetime v-model="dateTo" stack-label="Date de fin" />
      </div>
      <template slot="buttons" slot-scope="props">
        <q-btn flat label="Valider" color="primary" @click="props.ok"/>
      </template>
    </q-dialog>

    <!-- OVERLAY POUR LE FAB -->
    <template>
      <q-modal v-model="overlayFab" minimized class="my-modal" style="z-index: 1;">
      </q-modal>
    </template>

  </q-page>
</template>

<script>
import moment from 'moment'
import axios from 'axios'
import Vue from "vue"
import VueScrollPicker from "vue-scroll-picker"
import "vue-scroll-picker/dist/style.css"

Vue.use(VueScrollPicker)

export default {
  name: 'PageIndex',
  data () {
    return {
      poDisplayed: false,
      curSelDate: null,
      dateFrom: null,
      dateFromShort: null,
      dateTo: null,
      dateToShort: null,
      range: null,
      pickDateDialog: false,
      pickRangeDateDialog: false,
      queryPoList: {
        "size": 5000,
        "sort": [
          {
            "expected_date": {
              "order": "desc",
              "unmapped_type": "boolean"
            }
          }
        ],
        "_source": {
          "includes": [ 
            "number", 
            "status", 
            "type", 
            "expected_date", 
            "supplier",
            "line_items"
          ]
        },
        "query": {
          "bool": {
            "must": [
              {
                "range": {
                  "expected_date": {
                    "gte": "",
                    "lte": ""
                  }
                }
              }
            ]
          }
        }
      },
      poList: [],
      indice: 'purchase_order*',
      currentPurchaseOrder: null,
      originalPurchaseOrder: null,
      currentCart: null,
      columns: [
        { name: 'name', required: true, label: 'Produit', align: 'left', field: 'full_title', sortable: true, style: 'width: 200px' },
        { name: 'qty', required: true, label: 'Qté cdée', align: 'left', field: 'quantity', sortable: false, style: 'width: 40px' },
        { name: 'received', required: false, label: 'Qté reçue', align: 'left', field: 'received', sortable: false, style: 'width: 40px' },
        { name: 'action', required: false, label: 'Actions', align: 'right', field: 'action', sortable: false, style: 'width: 100px' }
      ],
      pagination: { rowsPerPage: 500 },
      problemBox: false,
      currentEditId: null,
      currentEditQty: null,
      currentEditQty_received: null,
      sendBtn: true,
      overlayFab: false,
      showTooltip: true,
      scrollPickerOptions: [1, 2, 3, 4, 5]
    }
  },
  methods: {
    getPoList () {
      //console.log(' >>>>> AVANT ENVOI : dateFrom/dateTo : ' + this.dateFrom + '/' + this.dateTo)
      this.isDateCorrect()
      this.queryPoList.query.bool.must[0].range.expected_date.gte = this.dateFrom
      this.queryPoList.query.bool.must[0].range.expected_date.lte = this.dateTo
      
      var url = this.$store.getters.apiurl + "generic_search/" +
        this.indice + "?token=" + this.$store.getters.creds.token
      //console.log('url : ', url)

      this.$q.loading.show()
      axios.post(url, this.queryPoList, { headers: { 'Content-Type': 'application/json' }} )
        .then( response => {
          this.poList = response.data.records 
          console.log("Data succesfully received : ", this.poList)
          this.$q.loading.hide()
        }).catch( error => {
          console.log('| getPoList / POST | UN PROBLEME EST SURVENU : ', error)
          this.$q.loading.hide()
        })
      
    },
    getPurchaseOrder (id, index) {
      //console.log(' ################ >> getPurchaseOrder [id: '+ id +'] & [index: '+ index +']')
    
      var url = this.$store.getters.apiurl + "generic/" + 
        index + "/" + id + "?token=" + this.$store.getters.creds.token
      //console.log('url: ', url)

      this.$q.loading.show()
      this.timer = setTimeout(() => { this.timer = void 0 }, 4500)
      axios.get(url)
        .then( response => {
          this.currentPurchaseOrder = response.data.data._source
          this.currentPurchaseOrder.line_items = Array.from(JSON.parse(this.currentPurchaseOrder.line_items))
          this.originalPurchaseOrder = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
          //console.log('test >>>>>>> ', JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder))
          for (var i =0; i < this.currentPurchaseOrder.line_items.length; i++) {
            this.currentPurchaseOrder.line_items[i].received = null
          }
          this.poDisplayed = true
          this.sendBtn = true
          this.$q.loading.hide()
        })
        .catch( error => {
          console.log('| getPurchaseOrder / GET| UN PROBLEME EST SURVENU : ', error)
          this.$q.loading.hide()
        })
        
    },
    onToday () {
      this.dateFrom = moment().startOf('day').unix()*1000
      this.dateTo = moment().endOf('day').unix()*1000
      this.dateFromShort = moment().format('DD-MM-YYYY')
      this.dateToShort = moment().format('DD-MM-YYYY')
      this.getPoList()
    },
    goodLookingDate (date) {
      moment.locale('fr')
      return moment(date).format('DD MMMM YYYY')
    },
    loadMore: function(index, done) {
      // index - called for nth time
      // done - Function to call when you made all necessary updates.
      //        DO NOT forget to call it otherwise your loading message
      //        will continue to be displayed. Has optional boolean
      //        parameter that invokes stop() when true

      // make some Ajax call then call done()
      // console.log("We want to load more stuff (index: " + index + ")")

      // setTimeout(() => {
      //   if (this.items) {
      //     this.items.push({}, {}, {}, {}, {}, {}, {})
      //     done()
      //   }
      // }, 25000)
      // //this.items.push({}, {}, {}, {}, {})
      // console.log('items contient maintenant ' + this.items.length + ' elements.')
    },
    onOkDatePicker () {
      this.dateFrom = moment(this.curSelDate).startOf('day').unix()*1000
      this.dateTo = moment(this.curSelDate).endOf('day').unix()*1000
      this.dateFromShort = moment(this.dateFrom).format('DD-MM-YYYY')
      this.dateToShort = moment(this.dateTo).format('DD-MM-YYYY')
      this.getPoList()
    },
    onOkRangeDatePicker () {
      this.dateFromShort = moment(this.dateFrom).format('DD-MM-YYYY')
      this.dateToShort = moment(this.dateTo).format('DD-MM-YYYY')
      this.getPoList()
    },
    backToList () {
      this.poDisplayed = false
      this.currentPurchaseOrder = null
    },
    checkNumber (str) {
      if (str === 'CREATED_BY_NYX') return 'Nyx'
      return str
    },
    chipStatus (str) {
      if (str === 'to_be_collected') return 'A VALIDER'
      if (str === 'fully_collected') return 'TERMINEE'
    },
    chipColor (str) {
      if (str === 'to_be_collected') return 'blue'
      if (str === 'fully_collected') return 'green'
    },
    lineIsOk (index) {
      //console.log('LINE IS OK >>>> ', this.currentPurchaseOrder.line_items)
      this.currentPurchaseOrder.line_items[index].received = this.currentPurchaseOrder.line_items[index].quantity
      //console.log('we want this line all ok', this.currentCart[index])
      var tmp = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
      this.currentPurchaseOrder = null
      this.currentPurchaseOrder = tmp
    },
    onCancelProblem () {
      this.currentEditQty = null
      this.currentEditQty_received = null
    },
    onOkProblem () {
      this.currentPurchaseOrder.line_items[this.currentEditId].received = this.currentEditQty_received
      this.currentEditQty = null
      this.currentEditQty_received = null
    },
    openProblemBox (id) {
      this.problemBox = true
      this.currentEditId = id
      this.currentEditQty = this.currentPurchaseOrder.line_items[id].quantity
      this.currentEditQty_received = this.currentPurchaseOrder.line_items[id].received
    },
    sendData () {
      console.log('======= SENDING DATA =========')
      console.log('order : ', this.currentPurchaseOrder)
      console.log('cart : ', this.currentCart)
      // modifier le status du bon
      this.$q.notify({ message: 'Bon de commande sauvegardé', timeout: 1000, color: 'green' })
    },
    // getSupplierName () {
    //   console.log('!!!!!!!!!!!! getgetget : ', this.currentPurchaseOrder)
    //   return this.currentPurchaseOrder
    // },
    getTheColor (obj, str) {
      switch (str) {
        case 'bg':
          if (obj.received === null) return
          else if (obj.received == obj.quantity) return 'bg-green-5'
          else return 'bg-red-5'
          break;
        case 'chip':
          if (obj.received === null) return 'grey-5'
          else if (obj.received == obj.quantity) return 'green-10'
          else return 'red-10'
          break;
        case 'btn':
          if (obj.received === null) return 'grey-5'
          else if (obj.received == obj.quantity) return 'green-10'
          else return 'red-10'
          break;
        case 'name':
          if (obj.received === null) return 'color: grey-5;'
          else return 'color: white;'
          break;
        default:
          console.log('Probleme switch getTheColor()');
      }
    },
    toggleFab () {
      this.showTooltip != this.showTooltip 
    },
    isDateCorrect () {
      if (isNaN(this.dateFrom) && isNaN(this.dateTo)) {
        this.dateFrom = moment().startOf('day').unix()*1000
        this.dateTo = moment().endOf('day').unix()*1000
        this.dateFromShort = moment().format('DD-MM-YYYY')
        this.dateToShort = moment().format('DD-MM-YYYY')
      }
    }
  },
  mounted () {
    this.onToday()
  },
  computed: {
    hasPoChanged () {
      return JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder)
    }
  },
  components: {
    
  },
  watch: {
    '$route' (to, from) {
      console.log(' =||=====||||=====||= What\'s happening here ?!?')
    }
}
}
</script>

<style>
  .my-card-list {
    /* display: block; */
    padding: 10px;
  }
  .date-ref {
    width: 100%;
  }
  .q-list-striped > .q-item:nth-child(even) {
    background-color: rgba(242, 240, 240, 0.65);
  }
  .my-modal {
    background-color: rgba(0, 0, 0, 0.562)
  }
  .my-dialog /deep/ .modal-content {
    width: 95vw;
    height: 90vh;
  }
  .my-fab-action {
    border-radius: 30px;
    width: 140px;
    height: 55px;
  }
  .q-fab-actions {
    margin-right: 185px;
  }
  i.q-icon {
    margin-right: 10px;
  }
  i.q-icon.q-fab-active-icon, 
  i.q-icon.q-fab-icon {
    margin-right: 0px;
  }
  .my-fab-action div:nth-child(2) {
    display: inline;
    float: left;
    padding: 15px;
  }

</style>