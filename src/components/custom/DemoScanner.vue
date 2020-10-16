<template>
  <!--
    _ boutons du q-fab (label + taille)
    _ overlay sous q-fab
    _ q-spinner-dots ??
    _ X chip "a valider", position
    _ datepicker taille dans dialog box
    _ X datepicker range ??
    _ X changer couleur de la chip
    _ envoi des données si retour a la liste ?
    _ actualisation de la liste quand bouton "pouce vert" clické !
    _ max height chip liste
    _ bouton retour -> envoi serveur
  -->
  <!-- 
    _ possibilité de trier les tuile dans un certain ordre
    _ repartir les tuiles par utilisateur
    _ ajout de produit depuis le BO (la ou reparti les utilisateurs)
    _liste produit => toute verte si tout ok, toute orange si incomplet, grise si pas encore remplie
        -> retire le chiffre "a verifier" on garde que la quantité
        engrenage -> pb & coche -> valid

  -->
  <q-page class="bg-grey-5">
    
<!-- SINGLE PURCHASE ORDER -->
    <div v-if="poDisplayed" class="q-pa-xs">
      <!-- <div class="text-center q-pa-sm">
        <q-btn color="primary" label="Retour" @click="backToList" />
      </div> -->
      
      <!-- CART / COLLAPSIBLE SUPPLIER INFOS -->
      <q-card flat bordered class="bg-white q-ma-md q-pa-md">
        <q-collapsible icon="perm_identity" :label="this.currentPurchaseOrder.supplier+' ('+goodLookingDate(this.currentPurchaseOrder.expected_date)+')'">
          <div>
            <div><span class="caption">Date commande :</span> <b>{{ goodLookingDate(this.currentPurchaseOrder.expected_date) }}</b></div>
            <div><span class="caption">Producteur :</span> <b>{{ goodLookingSupplier(this.currentPurchaseOrder.supplier) }}</b></div>
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
            <q-item :class="getTheColor(item, 'bg')" name="q-item-base" v-for="(item, index) in this.currentPurchaseOrder.line_items" :key="item.product_id" clickable v-ripple style="padding: 8px; min-height:60px;">
              <q-item-section color="yellow" name="item_name" style="max-width: 150px;">
                <q-item-label :style="getTheColor(item, 'name')" name="q-item-label">{{ item.full_title }}</q-item-label>
              </q-item-section>
              <div class="absolute-right">
                <q-item-section name="item_qty" class="q-mr-xs text-right" style="display: inline-block; min-width: 31px;">
                  <q-chip small pointing="right" style="width: 45px" :color="getTheColor(item, 'chip')">
                    <b>{{ item.quantity }}</b>
                  </q-chip>
                </q-item-section>
                <q-item-section v-model="item.received" name="item_qty_received" class="q-mr-xs text-center" style="display: inline-block; min-width: 31px;">
                  {{ item.received }}
                </q-item-section>
                <q-item-section name="icon_ok">
                  <q-btn size="30px" flat dense square icon="settings" :color="getTheColor(item, 'btn')" @click="openProblemBox(index)" />
                  <q-btn size="30px" flat dense square icon="done" :color="getTheColor(item, 'btn')" @click="lineIsOk(index)" />
                </q-item-section>
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
          <span class="on-left">0</span><span class="float-right">{{ currentEditQty }}</span>
          <q-slider
            v-model="currentEditQty_received"
            :min="0"
            :max="currentEditQty"
            :step="1"
            label
            snap
            markers
          />
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel" />
          <q-btn flat label="Valider" color="primary" @click="props.ok"/>
        </template>
      </q-dialog>

    </div>

    <div v-else>
<!-- PURCHASE ORDERS LIST -->
      <!-- REFERENCE DATE -->
      <div class="date-ref bg-grey-8 q-py-sm q-px-md text-white">
          <div v-if="this.dateFromShort === this.dateToShort" class="text-center">
            Date : {{ goodLookingDate(this.dateFrom) }}
          </div>
          <div v-else class="text-center">
            Période : du {{ goodLookingDate(this.dateFrom) }} au {{ goodLookingDate(this.dateTo) }}
          </div>
      </div>
      <!-- INFINITE SCROLL / PO LIST-->
      <q-infinite-scroll :handler="loadMore" v-if="poList.length">
        <div v-for="purchaseOrder in poList" :key="purchaseOrder.id" class="q-py-xs q-px-sm">
          <q-card class="bg-white">
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
      <q-fab-action color="primary" @click="onToday" icon="today" size="xl">
        <q-tooltip v-model="showTooltip" anchor="center left" self="center right" :offset="[20, 0]">
          Aujourd'hui
        </q-tooltip>
      </q-fab-action>

      <q-fab-action color="primary" @click="pickDateDialog = true" icon="event">
        Date
      </q-fab-action>

      <q-fab-action color="primary" @click="pickRangeDateDialog = true" icon="date_range">
        Période
      </q-fab-action>
    </q-fab>

    <!-- DATE PICKER DIALOG BOX -->
    <q-dialog v-model="pickDateDialog" prevent-close @cancel="onCancelDatePicker" @ok="onOkDatePicker">
        <span slot="title">Choisir un jour</span>
        <div slot="body">
          <q-datetime-picker v-model="curSelDate" type="date" />
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel" />
          <q-btn flat label="Valider" color="primary" @click="props.ok"/>
        </template>
    </q-dialog>

    <!-- RANGE DATE PICKER DIALOG BOX -->
    <q-dialog v-model="pickRangeDateDialog" prevent-close @cancel="onCancelRangeDatePicker" @ok="onOkRangeDatePicker">
        <span slot="title">Choisir une période</span>
        <div slot="body">
          <q-input v-model="dateFrom" float-label="Date début" placeholder="Gigi" />
          <q-input v-model="dateTo" float-label="Date fin" placeholder="Gigi" />
        </div>
        <template slot="buttons" slot-scope="props">
          <q-btn flat label="Annuler" color="primary" @click="props.cancel" />
          <q-btn flat label="Valider" color="primary" @click="props.ok"/>
        </template>
    </q-dialog>

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
      pagination: { rowsPerPage: 250 },
      fakeCart: [
        {'full_title':'Bave (170g)', 'quantity':12 },
        {'full_title':'Bave (170g)', 'quantity':154 },
        {'full_title':'Bavette de boeuf (170g)', 'quantity':3 },
        {'full_title':'Blanc de poulet (lot de 2x200g)', 'quantity':11 },
        {'full_title':'Blanquette de veau (500g)', 'quantity':1 },
        {'full_title':'Boudin noir (150g)', 'quantity':1 },
        {'full_title':'Boeuf bourguignon (1kg)', 'quantity':1 },
        {'full_title':'Chair à saucisse (250g)', 'quantity':3 },
        {'full_title':'Choucroute pour 2 personnes', 'quantity':1 },
        {'full_title':'Cordon bleu de dinde maison (lot de 2x170g)', 'quantity':2 },
        {'full_title':'Côte de porc (250g)', 'quantity':2 },
        {'full_title':'Entrecôte de boeuf (250g)', 'quantity':2 },
        {'full_title':'Escalope de dinde fermière (lot de 2x200g)', 'quantity':2 },
        {'full_title':'Escalope de veau (lot de 2x200g)', 'quantity':1 },
        {'full_title':'Filet mignon de porc (500g)', 'quantity':1 },
        {'full_title':'Jambon cru des Ardenne IGP (lot de 4 tranches x30g)', 'quantity':3 },
        {'full_title':'Jambon cuit (lot de 4 tranches x80g)', 'quantity':1 },
        {'full_title':'Lard fumé (500g)', 'quantity':1 },
        {'full_title':'Poire de boeuf (250g)', 'quantity':2 },
        {'full_title':'Rôti de boeuf Rumsteak (500g)', 'quantity':1 },
        {'full_title':'Saucisse de Montbelliard IGP (180g)', 'quantity':1 },
        {'full_title':'Saucisse de Toulouse (lot de 2x140g)', 'quantity':1 },
        {'full_title':'Saucisses chipolatas (lot de 4x80g)', 'quantity':3 },
        {'full_title':'Sauté de porc (1kg)', 'quantity':1 },
        {'full_title':'Terrine de campagne (180g)', 'quantity':2 },
        {'full_title':'Tranche de gigot (250g)', 'quantity':2 }
      ],
      problemBox: false,
      currentEditId: null,
      currentEditQty: null,
      currentEditQty_received: null,
      sendBtn: true,
      overlayFab: false,
      showTooltip: true
    }
  },
  methods: {
    getPoList () {
      console.log(' >>>>> AVANT ENVOI : dateFrom/dateTo : ' + this.dateFrom + '/' + this.dateTo)
      
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
          //this.$q.loading.hide()
        }).catch( error => {
          console.log('| getPoList / POST | UN PROBLEME EST SURVENU : ', error)
          
        })
      this.$q.loading.hide()
    },
    getPurchaseOrder (id, index) {
      console.log(' >>>>>>>>>>>> getPurchaseOrder id: ', id)
      console.log(' >>>>>>>>>>>> getPurchaseOrder index: ', index)

      this.poDisplayed = true
      this.sendBtn = true
      
      var url = this.$store.getters.apiurl + "generic/" + 
        index + "/" + id + "?token=" + this.$store.getters.creds.token
      //console.log('url: ', url)

      this.$q.loading.show()
      this.timer = setTimeout(() => { this.timer = void 0 }, 4500)
      axios.get(url)
        .then( response => {
          //console.log("response : ", response)
          this.currentPurchaseOrder = response.data.data._source
          console.log('toto', this.currentPurchaseOrder.line_items)
          this.currentPurchaseOrder.line_items = Array.from(JSON.parse(this.currentPurchaseOrder.line_items))
          //console.log('getPO  sans >>>>>>> ', this.currentPurchaseOrder)
          this.originalPurchaseOrder = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
          
          console.log('test >>>>>>> ', JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder))

          //console.log('getPO ori >>> ', this.originalCart)
          //console.log(' >>>>>>>>>>>> getPurchaseOrder successfully received data : ', this.currentPurchaseOrder)
          //console.log(' >>>>>>>>>>>> getPurchaseOrder panier inspection : ', this.currentCart)
          //this.currentCart.forEach(item => item['qty_received'] = '?')
          console.log(' ############ line items : ', this.currentPurchaseOrder.line_items)
          for (var i =0; i < this.currentPurchaseOrder.line_items.length; i++) {
            this.currentPurchaseOrder.line_items[i].received = null
          }
          
          
          
          //console.log('array from >> ', Array.from(this.currentPurchaseOrder.line_items))
          
          console.log(' >>>>>>>>>>>> getPurchaseOrder panier POST foreach : ', this.currentPurchaseOrder.line_items)
        })
        .catch( error => {
          console.log('| getPurchaseOrder / GET| UN PROBLEME EST SURVENU : ', error)
        })
        this.$q.loading.hide()
    },
    onToday () {
      this.dateFrom = moment().startOf('day').unix()*1000
      this.dateTo = moment().endOf('day').unix()*1000
      this.dateFromShort = moment().format('DD-MM-YYYY')
      this.dateToShort = moment().format('DD-MM-YYYY')
      this.getPoList()
    },
    onRange () {
      console.log('Button <range> clicked')
    },
    goodLookingDate (date) {
      moment.locale('fr')
      return moment(date).format('DD MMMM YYYY')
    },
    goodLookingSupplier(str) {
      const words = str.split(' - ')
      return words[1]
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
    onCancelDatePicker () {
    },
    onOkDatePicker () {
      this.dateFrom = moment(this.curSelDate).startOf('day').unix()*1000
      this.dateTo = moment(this.curSelDate).endOf('day').unix()*1000
      this.dateFromShort = moment().format('DD-MM-YYYY')
      this.dateToShort = moment().format('DD-MM-YYYY')
      this.getPoList()
    },
    onCancelRangeDatePicker () {
    },
    onOkRangeDatePicker () {
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
      console.log('LINE IS OK >>>> ', this.currentPurchaseOrder.line_items)
      this.currentPurchaseOrder.line_items[index].received = this.currentPurchaseOrder.line_items[index].quantity
      //console.log('we want this line all ok', this.currentCart[index])
      var tmp = JSON.parse(JSON.stringify(this.currentPurchaseOrder))
      this.currentPurchaseOrder = null
      this.currentPurchaseOrder = tmp
    },
    onCancelProblem () {
      this.currentEditQty = null
      this.currentEditQty_received = null
      //console.log('Josephine a dit /cancel/ !')
    },
    onOkProblem () {
      console.log('ON OK PROBLEM - ENTER')
      //this.currentCart[this.currentEditId].qty_received = this.currentEditQty_received
      this.currentPurchaseOrder.line_items[this.currentEditId].received = this.currentEditQty_received
      this.currentEditQty = null
      this.currentEditQty_received = null
      console.log('Josephine a dit /ok/ !', this.currentPurchaseOrder.line_items)
      //this.checkForFinishedOrder()
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
    getSupplierName () {
      console.log('getgetget : ', this.currentPurchaseOrder)
      return this.currentPurchaseOrder
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
      //console.log("AKUNA MATATA")
      //this.overlayFab = !this.overlayFab
      this.showTooltip != this.showTooltip 
    }
  },
  mounted () {
    this.dateFrom = moment().startOf('day').unix()*1000
    this.dateTo = moment().endOf('day').unix()*1000
    this.dateFromShort = moment().format('DD-MM-YYYY')
    this.dateToShort = moment().format('DD-MM-YYYY')
    this.getPoList()
  },
  computed: {
    hasPoChanged () {
      return JSON.stringify(this.currentPurchaseOrder) != JSON.stringify(this.originalPurchaseOrder)
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
</style>