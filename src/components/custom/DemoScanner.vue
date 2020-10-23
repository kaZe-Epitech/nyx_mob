<!-- to_be/fully/partially_collected -->
<!-- new chip system > etat - quantité -->

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
            <q-card-separator />
            <q-btn color="primary" class="full-width q-my-sm" label="Ajouter un produit" @click="addProduct" size="lg" />
          </q-card>
        </q-collapsible>

        <!-- CART ITEMS LIST -->
        <div>
          <q-list class="bg-white" separator bordered>
            <q-item :class="getTheColor(item, 'bg')" v-for="(item, index) in this.currentPurchaseOrder.line_items" :key="item.product_id" style="padding: 8px; min-height:60px;">
              <q-item-side style="max-width: 150px;">
                <div :style="getTheColor(item, 'name')" name="q-item-label">{{ item.full_title }}</div>
              </q-item-side>
              <q-item-main>
                <div class="q-mr-xs text-right" style="display: inline-block; min-width: 75px; float: right;">
                  <q-chip small style="width: 75px; text-align: center" :color="getTheColor(item, 'chip')">
                    <!-- <b>{{ item.received }} / {{ item.quantity }}</b> -->
                    <b>{{ chipFill(item.quantity, item.received) }}</b>
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
        <!-- BUTTON RETOUR -->
        <div class="q-mt-md text-center">
          <q-btn label="Valider" color="primary" icon-right="arrow_forward_ios" class="q-mx-lg" @click="backToList" />
          <!-- <q-btn label="Valider le bon" color="light-blue-2" icon-right="send" class="q-mx-lg" :disable="!hasPoChanged" @click="sendData" /> -->
        </div>
      </q-card>

      <!-- REPORT PROBLEM DIALOG BOX -->
      <q-dialog v-model="problemBox" @ok="onOkProblem" @cancel="onCancelProblem">
        <span slot="title">Quantité reçue</span>
        <div slot="body">
          <div class="flex-center row q-my-lg">
            <q-btn round color="primary" @click="onChangeQty(-1)" :repeat-timeout="100">
              <q-icon name="remove" />
            </q-btn>
            <q-input
              v-model="currentEditQty_received"
              align='center'
              hide-underline
              type="number"
              style="max-width: 50px; width: 50px;"
              :min="0"
              :max="currentEditQty" 
              class="q-mx-lg custom-input"
              @focus="focusField"
              @blur="blurField"
            />
            <q-btn round color="primary" @click="onChangeQty(1)" :repeat-timeout="100">
              <q-icon name="add" />
            </q-btn>
          </div>
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel"/>
          <q-btn flat label="Valider" color="primary" @click="props.ok"/>
        </template>
      </q-dialog>

      <!-- ADD A RAW PRODUCT -->
      <q-modal v-model="productSearch" maximized>
        <div style="padding: 15px">
          <div class="q-display-1 q-mb-md" style="max-width: 280px;">
            {{ currentPurchaseOrder.supplier }}
            <span class="absolute-top-right" style="margin: 15px;">
              <q-btn color="primary" @click="productSearch = false" label="Fermer" />
            </span>
          </div>
          <div>
            <div>
              <template v-if="noProducts">
                <q-item class="q-ma-lg q-pa-lg">
                  Aucun produit trouvé
                </q-item>
              </template>
              <template v-else>
                <q-list class="bg-white" separator bordered>
                  <q-item v-for="(item, index) in this.productList" :key="item.id" style="padding: 8px; min-height:60px;">
                    <q-item-main>
                      <div class="q-mr-xs text-right" style="display: inline-block; min-width: 75px; float: right;">
                        {{ item.title }}
                      </div>
                    </q-item-main>
                    <q-item-side>
                      <div>
                        <q-btn color="primary" round icon="add" @click="openRawQuantity(index)"  />
                      </div>
                    </q-item-side>
                  </q-item> 
                </q-list>
              </template>
            </div>
          </div>
        </div>
      </q-modal>

      <!-- QUANTITY BOX FOR RAW PRODUCT -->
      <!-- <q-modal v-model="rawQuantityBox" minimized>
        <div style="padding: 20px">
          <div class="text-center q-my-lg">
            <div class="q-display-1 q-mb-md">Quantité</div>
            <q-input
                v-model="rawQty"
                align='center'
                hide-underline
                type="number"
                style="max-width: 50px; width: 50px;"
                :min="1" 
                class="q-ma-lg custom-input"
                @focus="focusRawQty"
                @blur="blurRawQty"
              />
            <q-btn color="primary" v-close-overlay label="Valider" @click="addThisProduct()" />
          </div>
          
        </div>
      </q-modal> -->
      <q-dialog v-model="rawQuantityBox" @ok="addThisProduct()" @cancel="onCancelRaw">
        <span slot="title">Indiquez la quantité</span>
        <div slot="body">
          <div class="flex-center row q-my-lg">
            <q-btn round color="primary" @click="onChangeRawQty(-1)" :repeat-timeout="100">
              <q-icon name="remove" />
            </q-btn>
            <q-input
              v-model="rawQty"
              align='center'
              hide-underline
              type="number"
              style="max-width: 50px; width: 50px;"
              :min="0"
              class="q-mx-lg custom-input"
              @focus="focusRawQty"
              @blur="blurRawQty"
            />
            <q-btn round color="primary" @click="onChangeRawQty(1)" :repeat-timeout="100">
              <q-icon name="add" />
            </q-btn>
            <!-- TEST APPUI LONG -->
            <!-- <q-btn round color="primary" @click="clickHandler" :repeat-timeout="100">
              <q-icon name="add_circle_outline" />
            </q-btn> -->
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
      <q-infinite-scroll :handler="loadMore" v-if="poList.length" style="padding-bottom: 100px;">
        <div v-for="(purchaseOrder, index) in poList" :key="purchaseOrder.id" class="q-py-xs q-px-sm">
          <q-card class="bg-white" style="max-width: 700px;">
            <div v-ripple @click="getPurchaseOrder(purchaseOrder._id, purchaseOrder._index)" class="cursor-pointer relative-position ">
              <q-card-main style="padding: 10px; display: flex;">
                <div class="caption" style="width: 50px; margin-right: 5px;">
                  {{ purchaseOrder._source.expected_date | formatDate }}
                </div>
                <div class="caption" style="width: 220px;">
                  <b>{{ purchaseOrder._source.supplier }}</b>
                </div>
                <div class="caption" style="width: 80px;">
                  {{ checkNumber(purchaseOrder._source.number) }}
                </div>
                <q-chip dense :color="chipColor(index)" text-color="white" style="width: 140px; max-height: 25px;" class="text-center">
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

export default {
  name: 'PageIndex',
  // #region DATA  
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
      queryProductsList: {
        "size": 500,
        "sort": [
          {
            "updated_at": {
              "order": "desc",
              "unmapped_type": "boolean"
            }
          }
        ],
        "query": {
          "bool": {
            "must": [],
            "filter": [
              {
                "multi_match": {
                  "type": "phrase",
                  "query": "",
                  "lenient": true
                }
              }
            ]
          }
        }
      },
      poList: [],
      productList: [],
      indice: 'purchase_order*',
      indice2: 'product*',
      indice3: 'nyx_user',
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
      scrollPickerOptions: [],
      total: 0,
      tmpProp: null,
      minimizedModal: false,
      productSearch: false,
      noProducts: false,
      rawQuantityBox: false,
      rawQty: 1,
      rawIndex: null
    }
  },
  // #endregion 
  //#region METHODS 
  methods: {
    getPoList () {
      //console.log(' >>>>> AVANT ENVOI : dateFrom/dateTo : ' + this.dateFrom + '/' + this.dateTo)
      this.isDateCorrect()
      this.queryPoList.query.bool.must[0].range.expected_date.gte = this.dateFrom
      this.queryPoList.query.bool.must[0].range.expected_date.lte = this.dateTo

      var url = this.$store.getters.apiurl + "generic_search/" +
        this.indice + "?token=" + this.$store.getters.creds.token
      console.log('url : ', url)

      this.$q.loading.show()
      axios.post(url, this.queryPoList, { headers: { 'Content-Type': 'application/json' }} )
        .then( response => {
          this.poList = response.data.records

          // UPDATING DISPLAY TRICK
          var tmp = this.poList
          this.poList = null
          this.poList = tmp

          //console.log("Data succesfully received : ", this.poList)
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
          //console.log('test >>>>>>> ', JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder))
          //console.log(' ################ >> getPurchaseOrder',  this.rawResponse)

          // add line_items.received field if it doesn't exist yet
          for (var i =0; i < this.currentPurchaseOrder.line_items.length; i++) {
            if (this.currentPurchaseOrder.line_items[i].received == null)
              this.currentPurchaseOrder.line_items[i].received = -1
          }
          // add units_ordered & units_received if it doesn't exist yet
          if (this.currentPurchaseOrder.units_ordered == null)
            this.currentPurchaseOrder.units_ordered = -1
          if (this.currentPurchaseOrder.units_received == null)
            this.currentPurchaseOrder.units_received = -1
          console.log('POST RECEIVED PROCESSING', this.currentPurchaseOrder)

          this.originalPurchaseOrder = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
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

      // any changes ?
      console.log(' ///// >>> original PO : ', this.originalPurchaseOrder)
      console.log(' ////// >>> current PO : ', this.currentPurchaseOrder)

      if (!this.hasPoChanged) return
      console.log('======= SOMETHING HAS CHANGED => WE\'RE SENDING DATA =========')

      // count ordered & received items
      this.currentPurchaseOrder.units_received = this.totalItemsReceived()
      this.currentPurchaseOrder.units_ordered = this.totalItems()

      // apply new status
      this.currentPurchaseOrder.status = this.changeStatus()

      // Update "updated_at" field (2020-10-21T01:00:01.408567+02:00)
      this.currentPurchaseOrder.updated_at = moment().format("YYYY-MM-DDTHH:mm:ss.SSSSSSZ")

      // prepare the purchase order format
      var tmp = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
      tmp.line_items = JSON.stringify(tmp.line_items)

      // forge the query
      var updatedPurchaseOrder = {
        "_index": this.rawResponse.data.data._index,
        "_source": tmp,
        "_id": this.rawResponse.data.data._id
      }
      console.log("######### PRE SEND : ", updatedPurchaseOrder)

      // send the update request
      this.$store.commit({
        type: "updateRecord",
        data: updatedPurchaseOrder
      })

      // notify user
      this.$q.notify({ message: 'Bon de commande sauvegardé', timeout: 2000, color: 'green' })
    },
    getTheProducts () {
      var url = this.$store.getters.apiurl + "generic_search/" +
      this.indice2 + "?token=" + this.$store.getters.creds.token
      console.log('url : ', url)

      this.queryProductsList.query.bool.filter[0].multi_match.query = this.currentPurchaseOrder.supplier
      //console.log("Verification query products : ", this.queryProductsList)

      this.$q.loading.show()
      axios.post(url, this.queryProductsList, { headers: { 'Content-Type': 'application/json' }} )
        .then( response => {
          this.productList = response.data.records
          console.log("Data succesfully received : ", response.data.records)
          this.prepareProducts()
          this.$q.loading.hide()
        }).catch( error => {
          console.log('| getTheProducts / POST | UN PROBLEME EST SURVENU : ', error)
          this.$q.loading.hide()
        })        
    },
    addProduct () {
      this.getTheProducts()
      this.productSearch = true
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
      console.log('hello there')
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
      this.sendData()
      this.currentPurchaseOrder = null
      this.rawResponse = null
      this.poList = []
      this.productList = null
      this.noProducts = false
      this.getPoList()
    },
    checkNumber (str) {
      if (str === 'CREATED_BY_NYX') return 'Nyx'
      return str
    },
    chipStatus (str) {
      if (str === 'to_be_collected') return ''
      if (str === 'fully_collected') return 'COMPLETE'
      if (str === 'partially_collected') return 'INCOMPLETE'
    },
    chipColor (int) {
      // console.log('debug chipColor', this.poList)
      var str = this.getThePassage(int)     
      if (str === 'passage') return 'blue'
      if (str === 'fini') return 'green'
      if (str === 'entrain') return 'orange'
    },
    chipFill (qty, qty_rvd) {
      // if (isNaN(this.currentEditQty_received)) {
      //   return this.currentEditQty_received + ' / '+ this.currentEditQty
      // } else {
      //   return this.currentEditQty
      // }
      // ##############""
      // if (isNaN(qty_rvd)) {
      //   return qty_rvd +' / '+ qty
      // } else {
      //   return qty
      // }
      if (qty_rvd == -1) {
        return qty
      } else if (qty_rvd == qty) {
        return qty
      } else {
        return qty_rvd +' / '+ qty
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
      if (this.currentEditQty_received > this.currentEditQty)
        this.currentEditQty_received = this.currentEditQty
      if (this.currentEditQty_received <= 0)
        this.currentEditQty_received = 0
      
      this.currentPurchaseOrder.line_items[this.currentEditId].received = this.currentEditQty_received
      this.currentEditQty = null
      this.currentEditQty_received = null
      this.currentEditId = null
    },
    onCancelRaw () {
      this.rawQty = 1
    },
    openProblemBox (id) {
      this.currentEditId = id
      this.currentEditQty = this.currentPurchaseOrder.line_items[id].quantity
      this.currentEditQty_received = this.currentPurchaseOrder.line_items[id].received
      //this.fillScrollPickerOptions(this.currentEditQty)
      // var tmp = this.currentEditQty_received
      // this.currentEditQty_received = null
      // this.currentEditQty_received = tmp
      this.problemBox = true
    },
    // fillScrollPickerOptions (int) {
    //   this.scrollPickerOptions = []
    //   for (var i = 0; i < int+1; i++) {
    //     this.scrollPickerOptions.push(i)
    //   }
    //   //console.log("OPTIONS >>>> ", this.scrollPickerOptions)
    // },
    getTheColor (obj, str) {
      switch (str) {
        case 'bg':
          if (obj.received == -1) return
          else if (obj.received == obj.quantity) return 'bg-green-5'
          else return 'bg-red-5'
          break;
        case 'chip':
          if (obj.received == -1) return 'grey-5'
          else if (obj.received == obj.quantity) return 'green-10'
          else return 'red-10'
          break;
        case 'btn':
          if (obj.received === -1) return 'grey-5'
          else if (obj.received == obj.quantity) return 'green-10'
          else return 'red-10'
          break;
        case 'name':
          if (obj.received === -1) return 'color: grey-5;'
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
      //console.log('###### totalItems ######')
      var u = 0
      for(var i = 0; i < this.currentPurchaseOrder.line_items.length; i++) {
        u += this.currentPurchaseOrder.line_items[i].quantity
      }
      console.log('>>>>>> TOTAL : ', u)
      return u
    },
    totalItemsReceived () {
      //console.log('###### totalItems ######')
      var u = 0
      for(var i = 0; i < this.currentPurchaseOrder.line_items.length; i++) {
        if(this.currentPurchaseOrder.line_items[i].received > 0)
          u += this.currentPurchaseOrder.line_items[i].received
      }
      console.log('>>>>>> TOTAL RECEIVED : ', u)
      return u
    },
    anyChangeToSubmit () {
      // return false si pas de difference
      // return true si des differences sont trouvées
      for (var i = 0; i < this.currentPurchaseOrder.line_items.length; i++) {
        if (this.currentPurchaseOrder.line_items[i].received > -1)
          return true
      }
      return false
    },
    changeStatus () {
      if (this.currentPurchaseOrder.units_received == -1) return 'to_be_collected'
      else if (this.currentPurchaseOrder.units_received == 0) return 'partially_collected'
      else if (this.currentPurchaseOrder.units_received == this.currentPurchaseOrder.units_ordered) return 'fully_collected'
      else return 'partially_collected'
    },
    onChangeQty (i) {
      this.currentEditQty_received += i
      if (this.currentEditQty_received < 0)
        this.currentEditQty_received = 0
      if (this.currentEditQty_received > this.currentEditQty)
        this.currentEditQty_received = this.currentEditQty
    },
    focusField () {
      this.currentEditQty_received = ''
    },
    blurField () {
      if (this.currentEditQty_received === '')
        this.currentEditQty_received = this.currentPurchaseOrder.line_items[this.currentEditId].received
    },
    addThisProduct () {
      console.log('we want to add this product ... ', this.productList[this.rawIndex])
      console.log('... to this purchase order : ', this.currentPurchaseOrder)
      
      if (this.rawQty > 5000)
        this.rawQty = 5000
      if (this.rawQty <= 0)
        this.rawQty = 0
      
      // need to check if the product isn't already in cart
      if (this.alreadyInCart(this.productList[this.rawIndex].title)) {
        this.$q.notify({ message: 'Le produit est déjà dans le bon de commande !', timeout: 5000, color: 'orange' })
        return
      }
      

      this.currentPurchaseOrder.line_items.push({
        'full_title': this.productList[this.rawIndex].title,
        'product_id': '',
        'quantity': this.rawQty,
        'received': this.rawQty,
        'variant_id': ''
      })
      this.$q.notify({ message: 'Produit ajouté au bon de commande !', timeout: 5000, color: 'green' })

      this.rawQuantityBox = false
      this.rawIndex = null
      this.rawQty = 1
    },
    prepareProducts () {
      console.log("//// PREPARE_PRODUCTS nb de produits : ", this.productList.length)
      if (this.productList.length == 0) {
        this.noProducts = true
        return
      }

      var array = []
      for (var i = 0; i < this.productList.length; i++) {
        array[i] = this.productList[i]._source
      }
      //console.log("//// PREPARE_PRODUCTS : ", array)

      array.sort(function(a, b) {
        var nameA = a.title.toLowerCase(), nameB = b.title.toLowerCase()
        if (nameA < nameB) //sort string ascending
          return -1
        if (nameA > nameB)
          return 1
        return 0 //default return value (no sorting)
      })
      //console.log("//// PREPARE_PRODUCTS trié : ", array)
      this.productList = array
    },
    getThePassage (int) {
      //console.log('######### getThePassage for index ['+int+']') 
      var cart = Array.from(JSON.parse(this.poList[int]._source.line_items))
      //console.log('debug cart post array.from.parse : ', cart)   
      var foundQty = false
      var foundMinusOne = false
      for (var i = 0; i < cart.length; i++) {
        // console.log('line_items [ '+ cart[i].full_title +' : '+ cart[i].received +' ]')
        if (cart[i].received > -1) foundQty = true
        else if (cart[i].received === -1) foundMinusOne = true 
        else foundMinusOne = true 
      }
      // console.log('######### getThePassage for index ['+int+'] : foundQty['+ foundQty +'] & foundMinusOne['+ foundMinusOne +']')
      if (foundQty && foundMinusOne) return 'entrain'
      else if (!foundQty && foundMinusOne) return 'passage'
      else return 'fini'
    },
    openRawQuantity (id) {
      this.rawIndex = id
      this.rawQty = 1
      this.rawQuantityBox = true
    },
    onChangeRawQty (i) {
      this.rawQty += i
      if (this.rawQty < 1)
        this.rawQty = 1
      if (this.rawQty > 5000)
        this.rawQty = 5000
    },
    focusRawQty () {
      this.rawQty = ''
    },
    blurRawQty () {
      if (this.rawQty === '')
        this.rawQty = 1
    },
    alreadyInCart (str) {
      console.log('already In Cart / we re looking for :  ', str)
      console.log('already In Cart : ', this.currentPurchaseOrder.line_items)
      for(var i = 0; i < this.currentPurchaseOrder.line_items.length; i++)  {
        if (this.currentPurchaseOrder.line_items[i].full_title == str) 
          return true
      }
      return false
    }
  },
  // #endregion
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
  .my-dialog .modal-content {
    width: 95vw;
    height: 90vh;
  }
  .my-fab-action {
    border-radius: 30px;
    width: 160px;
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
  .custom-input {
    height: 50px;
    font-size: 32px;
  }
</style>