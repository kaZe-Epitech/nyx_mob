<template>
  <div class="bg-grey-2">
    <q-page>
      <div class="q-pa-md q-gutter-md">

        <!-- PURCHASE ORDER LIST -->
        <div v-if="!poDisplayed">
          <!-- 3 buttons debug-->
          <div class="q-mb-md text-center">
            <q-btn color="primary" label="aujourd'hui" class="q-mr-md" @click="pickToday" />
            <q-btn color="primary" label="Choisir un jour" class="q-mr-md" @click="pickDate" />
            <q-btn color="primary" label="Choisir une période" class="q-mr-md" @click="pickRangeDate" />
          </div>
          <div v-for="(purOrd, index) in poList" :key="index" class="caption">      
            <q-card name="itemList" flat bordered class="q-pa-md row bg-white q-mb-sm" onClick="showPo">
              <div v-ripple @click="showPo(purOrd._id)" class="cursor-pointer relative-position">
                <q-card-section side name="itemList-date"> 
                  {{ goodLookingDate(purOrd._source.expected_date) }}
                </q-card-section>
                <q-card-section class="on-right" name="itemList-fournisseur"> 
                  {{ goodLookingSupplier(purOrd._source.supplier) }}
                </q-card-section>
                <q-card-section side v-show="dataEnabled" name="itemList-allData">
                  {{ index }} - {{ purOrd._id }} - {{ purOrd._source.expected_date }} - 
                  {{ purOrd._source.number }} - {{ purOrd._source.status }} - 
                  {{ purOrd._source.supplier }} - {{ purOrd._source.type }}
                </q-card-section>
              </div> 
            </q-card>
          </div>
        </div>

        <!-- SINGLE PURCHASE ORDER -->
        <div v-if="poDisplayed">
          <div class="q-mb-md">
            <q-btn color="primary" label="Retour" @click="goBack" />
          </div>
          
          <q-card flat bordered class="my-card bg-white">
            <p>
              {{ goodLookingDate(this.currentPurchaseOrder.expected_date) }}
            </p>
            <p>
              {{ goodLookingSupplier(this.currentPurchaseOrder.supplier) }}
            </p>
            <p>
              {{ this.currentPurchaseOrder.supplier }}
            </p>
            <p>
              {{ this.currentPurchaseOrder.number }}
            </p>
            <p>
              {{ this.currentPurchaseOrder.status }}
            </p>
            <p>
              {{ this.currentPurchaseOrder.type }}
            </p>
            <p>
              Nb d'items dans le panier : {{ countItems() }}
            </p>
            <!-- CART ITEM LIST -->
            <q-list class="bg-white" separator bordered> 
              <q-item name="q-item-base" v-for="(item, index) in JSON.parse(this.currentPurchaseOrder.line_items)" :key="index" clickable v-ripple>
                <q-item-section name="item_index" class="caption">
                  <q-item-label>{{ index }} - </q-item-label>
                </q-item-section>
                <q-item-section name="item_name">
                  <q-item-label>{{ item.full_title }} - </q-item-label>
                </q-item-section>
                <q-item-section name="item_qty">
                  {{ item.quantity }}
                </q-item-section>
                <q-item-section name="icon_ok" side>
                  <div class="text-green q-gutter-lg">
                    <q-btn size="20px" flat dense square icon="thumb_up_alt" color="green" />
                  </div>
                </q-item-section>
                <q-item-section name="icon_pb" side>
                  <div class="text-red q-gutter-lg">
                    <q-btn size="20px" flat dense square icon="report_problem" color="red" />
                  </div>
                </q-item-section>
              </q-item>
            </q-list>

            <!-- REPORT PROBLEM DIALOG BOX -->
            <q-dialog v-model="problemBox" persistent>
              <q-card style="width: 300px" class="q-px-sm q-pb-md">
                <q-card-section>
                  <div class="text-h6">Quantité reçue</div>
                  <q-item dense>
                    <q-item-section avatar>
                      <q-icon name="qtity" />
                    </q-item-section>
                    <q-item-section>
                      <q-slider color="teal" v-model="slideQuantity" :step="1" />
                    </q-item-section>
                  </q-item>
                </q-card-section>
                <q-card-actions align="right">
                  <q-btn flat label="Annuler" v-close-popup />
                  <q-btn flat label="Valider" v-close-popup />
                </q-card-actions>
              </q-card>
            </q-dialog>

          </q-card>
        </div>
      
      </div>
      
      <!-- STICKY FLOATING BUTTON -->
      <!-- <div position="bottom-right" :offset="[15, 15]">
        <q-fab vertical-actions-align="right" color="primary" text-color="white" icon="calendar_today" direction="up">
          <q-fab-action label-position="left" color="primary" @click="today" icon="today" label="ajourd'hui" />
        </q-fab>
      </div> -->

      <!-- DATE PICKER DIALOG BOX -->
      <q-dialog v-model="pickDateDialog" persistent>
        <q-card class="q-pa-md q-ma-md bg-white">
          <q-card-section class="row items-center">
            <q-avatar icon="event_available" color="primary" text-color="white" />
            <span class="q-ml-sm">Choisir une date</span>
          </q-card-section>
          <!-- DATE PICKER-->
          <q-card-section>
            <q-date v-model="date" />
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="Annuler" color="primary" v-close-popup />
            <q-btn flat label="Valider" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <!-- RANGE DATE PICKER DIALOG BOX -->
      <q-dialog v-model="pickRangeDateDialog" persistent>
        <q-card class="q-pa-md q-ma-md bg-white">
          <q-card-section class="row items-center">
            <q-avatar icon="event_available" color="primary" text-color="white" />
            <span class="q-ml-sm">Choisir une période</span>
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="Annuler" color="primary" v-close-popup />
            <q-btn flat label="Valider" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>

    </q-page>
  </div>
</template>

<script>
import moment from 'moment'
import axios from 'axios'

export default {
  name: 'PageIndex',
  data () {
    return {
      indice: 'purchase_order',
      now: null,
      queryList: {
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
            "supplier" 
          ]
        },
        "query": {
          "bool": {
            "must": [
              {
                "range": {
                  "expected_date": {
                    "gte": "2020-10-01",
                    "lte": "2020-10-10"
                  }
                }
              }
            ]
          }
        }
      },
      poList: null,
      dateFrom: null,
      dateTo: null,
      dataEnabled: true, // A RETIRER AINSI QUE LA CARD-SECTION DANS *PURCHASE ORDER LIST*
      poDisplayed: false,
      currentPurchaseOrder: null,
      problemBox: false,
      slideQuantity: 0,
      pickDateDialog: false,
      pickRangeDateDialog: false
    }
  },
  methods: {
    getPoList () {
      var url = this.$store.getters.apiurl + "generic_search/" +
        "purchase_order" + "?token=" + this.$store.getters.creds.token

      axios.post(url, this.queryList, { headers: { 'Content-Type': 'application/json' }} )
        .then( response => {
          this.poList = response.data.records        })
        .catch( error => {
          console.log('|POST| UN PROBLEME EST SURVENU : ', error)
        })
    },
    getPurchaseOrder (id) {
      var url = this.$store.getters.apiurl + "generic/" + 
      this.indice + "/" + id + "?token=" + this.$store.getters.creds.token

      axios.get(url)
        .then( response => {
          this.currentPurchaseOrder = response.data.data._source
          var panier = JSON.parse(this.currentPurchaseOrder.line_items)
        })
        .catch( error => {
          console.log('|REQ.GET| UN PROBLEME EST SURVENU : ', error)
        })
    },
    pickToday () {
      this.queryList.query.bool.must[0].range.expected_date.gte = this.now
      this.queryList.query.bool.must[0].range.expected_date.lte = this.now
      getPoList()
    },
    // pickDate() {
    //   this.pickDateDialog = true
    // },
    pickRangeDate() {
      this.pickRangeDateDialog = true
    },
    goodLookingDate (date) {
      moment.locale('fr')
      return moment(date).format('D MMMM YYYY')
    },
    goodLookingSupplier(str) {
      const words = str.split(' - ')
      return words[1]
    },
    showPo (id) {
      this.poDisplayed = true
      this.getPurchaseOrder(id)
    },
    goBack () {
      this.currentPurchaseOrder = null
      this.poDisplayed = !this.poDisplayed
    },
    countItems () {
      return JSON.parse(this.currentPurchaseOrder.line_items).length
    },
    reportProblem () {
      this.problemBox = true
    },
    pickDate () {
      this.$q.dialog({
          title: 'Please tell me :',
          message: 'Would you like a nice cup of tea?',
          cancel: true,
          persistent: true
        }).then(() => {
          console.log('>>>> OK')
        }).catch(() => {
          console.log('>>>> Cancel')
        })
    }
  },
  mounted () {
    var now = moment().format('YYYY-MM-DD')    
    this.getPoList()
  }
}
</script>

<style>
  .my-card {
    width: 100%;
    max-width: 250px;
  }
</style>
