<template>
  <div class="page">
    <!-- CONTENT VIEW -->
    <div class="container menu">
      <div class="columns columns-item-title column is-mobile title-main mg-b-0">
        <div class="column pd-r-0 pd-t-0 pd-b-0">
          <h1 class='text-align-left'>ข้อมูลจัดส่ง</h1>
        </div>
      </div>
      <section class="section display pd-l-0 pd-r-0">
        <div class="container-input">
          <p style="text-align:left">
            ชื่อ-นามสกุล <span style="color:red;">*</span>
          </p>
          <input class="input" v-model="address.name" placeholder="กรอกชื่อและนามสกุล" rows="1" type="text" />
        </div>

        <div class="container-input">
          <p style="text-align:left">
            เบอร์โทรศัพท์ <span style="color:red;">*</span>
          </p>
          <input class="input" type="tel" pattern="[0-9-]*" inputmode="numeric" v-model="address.phone" placeholder="กรอกเบอร์โทรศัพท์ที่สามารถติดต่อได้" rows="1" v-mask="'##-####-####'" />
        </div>

        <div class="container-input minimal">
          <label class="radio" v-for="(type, index) in order_type" :key="index" v-if='type.enable'>
            <input type="radio" name="showInput" v-model="selected_order_type" :value="index" @click="onChangedOrderType(index)">
            <span>{{ order_type_value[type.name].name }}</span>
          </label>
        </div>

        <div v-if="order_type.length > 0 && order_type[selected_order_type].name === 'delivery'">
          <div v-if="locate.length === 0">
            <div class="container-input minimal">
              <p style="text-align:left">
                บ้านเลขที่ <span style="color:red;">*</span>
              </p>
              <input class="input" v-model="address.locat" placeholder="บ้านเลขที่" rows="1" />
            </div>
            <div class="container-input minimal">
              <p style="text-align:left">
                อาคาร/หมู่บ้าน <span style="color:red;">*</span>
              </p>
              <input class="input" v-model="address.village" placeholder="อาคาร/หมู่บ้าน" rows="1" />
            </div>
            <div class="container-input minimal">
              <p style="text-align:left">
                ซอย
              </p>
              <input class="input" v-model="address.alley" placeholder="ซอย" rows="1" />
            </div>
            <div class="container-input minimal">
              <p style="text-align:left">
                รายละเอียด
              </p>
              <textarea class="textarea" v-model="address.detail" placeholder="รายละเอียดการเดินทาง เช่น ชั้น หรือเลขที่ห้อง" rows="5" />
            </div>
            <section class="section-map text-align-center">
              <div class="showInfo-map">
                  <div><img src="../assets/images/home.png" alt="" width="50%"></div>
                  <div class="text-map"><h3>ยังไม่มีตำแหน่งจัดส่งบนแผนที่</h3></div>
                  <router-link :to="{ name: 'googlemap' }">
                    <button class="btn button-map" type="button">ระบุตำแหน่ง</button>
                  </router-link>
              </div>
            </section>
            <!-- <div class="container-input" v-if="editMode" style="margin-top: 5%; text-align: center;">
              <label style="padding: 3%;">
                <button class="btn-cancle" type="button" v-on:click="cancelEditAddress()">
                  ยกเลิก
                </button>
              </label>=
            </div> -->
          <!-- <div style="padding: 3%;">
            <router-link :to="{ name: 'addAddress' }">
              <button class="btn" type="button">
                เพิ่มที่อยู่&nbsp;
                <i class="fas fa-plus fa-lg"></i>
              </button>
            </router-link>
          </div> -->
          </div>
          <div v-else>
            <div class="container-input minimal" v-for="(here, index) in locate" :key="index">
            <div class="col-md-4 col-lg-4 col-sm-4">
              <label class="location-label">
                <input class="box-shadow card-input-element" type="radio" name="product">

                <div class="panel panel-default card-input" style="border: 1px solid lightgrey; width:100%; border-radius: 4px;">
                  <div class="card-detail" style="background-color: white;">
                    <p style="text-align:left">บ้านเลขที่&nbsp;{{ here.locat }}&nbsp;อาคาร/หมู่บ้าน&nbsp;{{ here.village }}</p>
                    <p style="text-align:left">ซอย&nbsp;{{ here.alley }}</p>
                    <p style="text-align:left">{{ here.detail }}</p>
                  </div>
                  <div style="margin-left: 75%; margin-bottom: 2%;">
                    <button class="btn-del" v-on:click="deleteAddress(here)">
                      ลบ
                    </button>
                    <button class="btn-edit" v-on:click="editAddress(here)">
                      แก้ไข
                    </button>
                  </div>
                </div>
              </label>
            </div>
          </div>
        </div>
        </div>
      </section>
      <div class="footer-space" />
    </div>

    <!-- FOOTER -->
    <footer class="footer">
      <div class="content has-text-centered">
        <div class="summary">
          <div class='summary-total'>฿{{ subtotal.toFixed(2) }}</div>
          <router-link :to="{ name: 'summary' }">
            <button class="submit-order" v-on:click="saveData()" :disabled="address.name === '' || address.phone === ''">
              ถัดไป
            </button>
          </router-link>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
import Vue from 'vue'

export default {
  data () {
    return {
      selected_address: {},
      docID: '',
      locate: '',
      name: '',
      phone: '',
      defaultAdress: 0,
      coordinates: null,
      map_address: [],
      address: {
        name: '',
        phone: '',
        locat: '',
        village: '',
        alley: '',
        detail: ''
      },
      order_type: [],
      order_type_value: Vue.prototype.order_type_value,
      selected_order_type: -1,
      subtotal: 0,
      tmpAddress: '',
      editMode: false
    }
  },
  created () {
    console.log('selected_address:', this.selected_address)
    this.address = this.$store.state.address
    this.subtotal = this.$store.state.subtotal

    this.map_address = this.$store.state.map_address
    this.coordinates = this.$store.state.coordinates

    this.$http.get('https://asia-east2-testlocall.cloudfunctions.net/user', {params:
    { email: 'tew2@email', lineId: 'tew' }
    }).then((res) => {
      this.docID = res.data.docsId
      if (res.status !== 404) { 
        this.address.name = res.data.data.name
        this.locate = this.$store.state.addresses.length === 0 ? res.data.data.addresses : this.$store.state.addresses
        console.log('locate: ', this.locate.length)
        this.selected_address = this.locate[0]
        this.address.phone = res.data.data.phones[Number(res.data.data.defaultPhone)]
        this.defaultAdress = res.data.data.defaultAdress
        console.log('defaultAdress:', this.defaultAdress)
      }
    })
    this.order_type = this.$store.state.order_type
    if (this.order_type.length > 0) {
      if (this.$store.state.selected_order_type === -1) {
        this.selected_order_type = 0
        this.$store.commit('setSelectedOrderType', 0)
      } else {
        this.selected_order_type = this.$store.state.selected_order_type
      }
    }
    if (this.address.name === '' || this.address.name === undefined || this.address.name == null) {
      // Open in LINE LIFF
      if (this.liff.isInClient()) {
        this.$http.get('https://asia-east2-testlocall.cloudfunctions.net/user', {params:
        { lineId: this.liff.getDecodedIDToken().sub }
        }).then((res) => {
          console.log(res)
          if (res.status !== 404) {
            this.address.name = res.data.data.name
            this.locate = this.$store.state.addresses.length === 0 ? res.data.data.addresses : this.$store.state.addresses
            this.address.phone = res.data.data.phones[Number(res.data.data.defaultPhone)]
          }
        })
        this.liff.getProfile().then(profile => {
          this.address.name = profile.displayName
        })
      }
    }
  },
  methods: {
    saveData: function () {
      if (this.locate.length === 0) {
        var addresses = this.locate.concat([{
          'detail': this.address.detail,
          'village': this.address.village,
          'locat': this.address.locat,
          'alley': this.address.alley
        }])
        this.$http.post('https://asia-east2-testlocall.cloudfunctions.net/user/update', {
          'idFireStore': this.docID,
          'dataForUpdate': {'addresses': addresses}
        })
        this.$store.commit('setAddress', this.address)
      } else {
        this.address.detail = this.selected_address.detail
        this.address.village = this.selected_address.village
        this.address.locat = this.selected_address.locat
        this.address.alley = this.selected_address.alley
        this.$store.commit('setAddress', this.address)
      }
    },
    onChangedOrderType: function (index) {
      this.selected_order_type = index
      this.$store.commit('setSelectedOrderType', index)
    },
    deleteAddress: function (here) {
      var index = this.locate.indexOf(here)
      if (index > -1) {
        this.locate.splice(index, 1)
      }
      this.$http.post('https://asia-east2-testlocall.cloudfunctions.net/user/update', {
        'idFireStore': this.docID,
        'dataForUpdate': {'addresses': this.locate}
      })
    },
    editAddress: function () {
      // เพิ่มตัวแปร tmpAddress 
      this.editMode = true
      console.log(this.selected_address)
      this.tmpAddress = this.locate
      this.locate = []
      this.address.detail = this.selected_address.detail
      this.address.village = this.selected_address.village
      this.address.locat = this.selected_address.locat
      this.address.alley = this.selected_address.alley
    },
    cancelEditAddress: function () {
      this.locate = this.tmpAddress
      this.selected_address = this.locate[0]
      this.address.detail = this.selected_address.detail
      this.address.village = this.selected_address.village
      this.address.locat = this.selected_address.locat
      this.address.alley = this.selected_address.alley
    },
    addAddress: function () {
      var addresses = this.locate.concat([{
        'detail': this.address.detail,
        'village': this.address.village,
        'locat': this.address.locat,
        'alley': this.address.alley
      }])
      this.$http.post('https://asia-east2-testlocall.cloudfunctions.net/user/update', {
        'idFireStore': this.docID,
        'dataForUpdate': {'addresses': addresses}
      })
      this.$store.commit('setAddress', this.address)
    }
  }
}
</script>
