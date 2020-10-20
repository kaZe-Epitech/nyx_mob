<template>
  <q-page class="bg-grey-5">
    
<!-- SINGLE PURCHASE ORDER -->
    <div v-if="poDisplayed" class="q-pa-xs">
      <!-- CART / COLLAPSIBLE SUPPLIER INFOS -->
      <q-card flat bordered class="bg-white q-ma-xs q-pa-xs">
        <q-collapsible icon="perm_identity" :label="this.currentPurchaseOrder.supplier">
          <q-card class="q-pa-xs">
            <div><span class="caption">Date commande :</span> <b>{{ goodLookingDate(this.currentPurchaseOrder.expected_date) }}</b></div>
            <div><span class="caption">Producteur :</span> <b>{{ this.currentPurchaseOrder.supplier }}</b></div>
            <div><span class="caption">Contact : </span> <b>{{ this.currentPurchaseOrder.supplier }}</b></div>
            <div><span class="caption">Numéro de commande :</span> <b>{{ this.currentPurchaseOrder.number }}</b></div>
            <div><span class="caption">Statut actuel :</span> <b>{{ this.currentPurchaseOrder.status }}</b></div>
            <div><span class="caption">Type de bon :</span> <b>{{ this.currentPurchaseOrder.type }}</b></div>
            <div><span class="caption">Nb d'items :</span> <b>{{ this.currentPurchaseOrder.line_items.length }}</b></div>
            <div><span class="caption">Nb total de produits :</span> <b>{{ this.total }}</b></div>
          </q-card>
        </q-collapsible>      
             
        <!-- CART ITEMS LIST -->
        <div class="">
          <q-list class="bg-white" separator bordered>
            <q-item :class="getTheColor(item, 'bg')" v-for="(item, index) in this.currentPurchaseOrder.line_items" :key="item.product_id" style="padding: 8px; min-height:60px;">
              <q-item-side style="max-width: 150px;">
                <div :style="getTheColor(item, 'name')" name="q-item-label">{{ item.full_title }}</div>
              </q-item-side>
              <q-item-main>
                <div class="q-mr-xs text-right" style="display: inline-block; min-width: 75px; float: right;">
                  <q-chip small style="width: 75px; text-align: center" :color="getTheColor(item, 'chip')">
                    <b>{{ item.received }} / {{ item.quantity }}</b>
                    <!-- <b>{{ chipFill(item.quantity, item.received) }}</b> -->
                  </q-chip>
                </div> 
              </q-item-main>
              <q-item-side>
                <div>
                  <!-- old icon : settings -->
                  <q-btn size="20px" flat dense square icon="priority_high" :color="getTheColor(item, 'btn')" @click="openProblemBox(index)" class="btn-po" />
                  <q-btn size="20px" flat dense square icon="done" :color="getTheColor(item, 'btn')" @click="lineIsOk(index)" class="btn-po" />
                </div>
              </q-item-side>
            </q-item>          
          </q-list>
        </div>
        <!-- BOUTTONS RETOUR & SEND DATA -->
        <div class="q-mt-md text-center">
          <q-btn label="Retour" color="primary" icon="arrow_back_ios" class="q-mx-lg" @click="backToList" />
          <!-- <q-btn label="Valider le bon" color="light-blue-2" icon-right="send" class="q-mx-lg" :disable="!hasPoChanged" @click="sendData" /> -->
        </div>
      </q-card>

      <!-- REPORT PROBLEM DIALOG BOX -->
      <q-dialog v-if="problemBox" v-model="problemBox" @ok="onOkProblem" @cancel="onCancelProblem">
        <span slot="title">Quantité reçue</span>
        <div slot="body">
          <span class="on-left">0</span><span class="float-right">{{ currentEditQty }}</span>
          <q-slider v-model="currentEditQty_received" :min="0" :max="currentEditQty" :step="1" label snap markers />
          <div>
            <scroll-picker
              :options="scrollPickerOptions"
              v-model="currentEditQty_received"
              :drag-sensitivity="0.5"
              :touch-sensitivity="0.5"
              :scroll-sensitivity="0.5" 
              :value="0"
            />
          </div>
          
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel"/>
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
      <q-fab-action color="primary" @click="onToday" class="my-fab-action" icon="">
        Aujourd'hui
        <q-icon name="today" style="margin-left: 5px;" />
      </q-fab-action>
      <!-- DATE BUTTON -->
      <q-fab-action color="primary" @click="pickDateDialog = true" class="my-fab-action" icon="">
        Date
        <q-icon name="event" style="margin-left: 5px;" />
      </q-fab-action>
      <!-- RANGE DATE BUTTON -->
      <q-fab-action color="primary" @click="pickRangeDateDialog = true" class="my-fab-action" icon="">
        Période
        <q-icon name="date_range" style="margin-left: 5px;" />
      </q-fab-action>
    </q-fab>

    <!-- DATE PICKER DIALOG BOX -->
    <q-dialog v-model="pickDateDialog" @ok="onOkDatePicker">
      <span slot="title">Choisir un jour</span>
      <div slot="body">
        <q-datetime-picker v-model="curSelDate" format="DD/MM/YYYY" />
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
      indice2: 'nyx_user',
      rawResponse: null,
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
      currentEditQty_received: 0,
      sendBtn: true,
      overlayFab: false,
      showTooltip: true,
      scrollPickerOptions: [1,2,3,4,5,6],
      scrollVar: 1,
      total: 0
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
      // this.timer = setTimeout(() => { this.timer = void 0 }, 4500)
      axios.get(url)
        .then( response => {
          console.log(' #####  AXIOS.POST #####')
          this.rawResponse = response
          this.currentPurchaseOrder = response.data.data._source
          this.currentPurchaseOrder.line_items = Array.from(JSON.parse(this.currentPurchaseOrder.line_items))
          this.originalPurchaseOrder = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
          //console.log('test >>>>>>> ', JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder))
          //console.log(' ################ >> getPurchaseOrder',  this.rawResponse)         
          for (var i =0; i < this.currentPurchaseOrder.line_items.length; i++) {
            if (this.currentPurchaseOrder.line_items[i].received == null)
              this.currentPurchaseOrder.line_items[i].received = 0
          }
          this.poDisplayed = true
          this.sendBtn = true
          this.total = this.totalItems()
          this.$q.loading.hide()
        })
        .catch( error => {
          console.log('| getPurchaseOrder / GET| UN PROBLEME EST SURVENU : ', error)
          this.$q.loading.hide()
        }) 
    },
    sendData () {
      console.log('======= SENDING DATA =========')
      // console.log('order : ', this.currentPurchaseOrder)
      // console.log('cart : ', this.currentCart)

      // var updatedPurchaseOrder = {
      //   "_index": index.de.la.fiche,
      //   "_source": JSON.stringify(this.currentPurchaseOrder),
      //   "_id": id.de.la.fiche
      // }

      // this.$store.commit({
      //   type: "updateRecord",
      //   data: updatedPurchaseOrder
      // })

      console.log("######### PRE SEND : ", tmp)

      // modifier le status du bon
      this.$q.notify({ message: 'Bon de commande sauvegardé', timeout: 1000, color: 'green' })
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
    chipFill (qty, qty_rvd) {
      // if (isNaN(this.currentEditQty_received)) {
      //   return this.currentEditQty_received + ' / '+ this.currentEditQty
      // } else {
      //   return this.currentEditQty
      // }
      if (isNaN(qty_rvd)) {
        return qty_rvd +' / '+ qty
      } else {
        return qty
      }
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
      this.currentEditId = null
    },
    onOkProblem () {
      console.log("coucou ici")
      this.currentPurchaseOrder.line_items[this.currentEditId].received = this.currentEditQty_received
      this.currentEditQty = null
      this.currentEditQty_received = null
      this.currentEditId = null
    },
    openProblemBox (id) { 
      

      this.currentEditId = id
      this.currentEditQty = this.currentPurchaseOrder.line_items[id].quantity
      this.currentEditQty_received = this.currentPurchaseOrder.line_items[id].received
      
      // var tmp = this.currentEditQty_received
      // this.currentEditQty_received = null
      // this.currentEditQty_received = tmp

      //this.fillScrollPickerOptions(this.currentEditQty) 
      this.problemBox = true
    },
    fillScrollPickerOptions (int) {
      this.scrollPickerOptions = []
      for (var i = 0; i < int+1; i++) {
        this.scrollPickerOptions.push(i)
      }
      //console.log("OPTIONS >>>> ", this.scrollPickerOptions)
    },
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
    },
    totalItems () {
      console.log('###### totalItems ######')
      var u = 0
      for(var i = 0; i < this.currentPurchaseOrder.line_items.length; i++) {
        u += this.currentPurchaseOrder.line_items[i].quantity
      }
      console.log('>>>>>> TOTAL : ', u)
      return u
    }
  },
  mounted () {
    this.onToday()
  },
  computed: {
    hasPoChanged () {
      return JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder)
    }
  }
}
</script>

<style>
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
    width: 150px;
    height: 55px;
  }
  .q-fab-actions {
    margin-right: 185px;
  }
  .my-fab-action div:nth-child(2) {
    display: block;
    float: right;
    padding: 15px;
    text-align: right;
  }

</style>