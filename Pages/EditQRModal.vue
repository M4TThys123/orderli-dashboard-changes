<template>
  <modal v-if="qr" :show="show" v-on:update:show="onClose">
    <template slot="header">
      <h5 v-if="mode === 'edit'" class="modal-title">Editing QR</h5>
      <h5 v-else-if="mode === 'bulk'" class="modal-title">Bulk editing QRs</h5>
      <h5 v-else class="modal-title">New QR</h5>
    </template>
    <div class="mb--4">
      <form @keyup.enter.prevent="onsubmit" @submit.prevent="onsubmit">
        <div>
          <div class="row">
            <div class="col-lg-6">

              <base-input
                  type="text"
                  :disabled="mode === 'edit'"
                  name="name"
                  :ref="mode !== 'bulk' ? 'autofocus' : 'no-autofocus'"
                  required
                  label="QR Code"
                  v-model="qr.code"
              >
                <div v-if="mode === 'bulk' && !bulkFields.includes('code')" slot="appendLabel" class="bulkEdit" @click="enableField('code', $event)">Edit</div>
              </base-input>

            </div>
            <div class="col-lg-6">
              <base-input
                type="text"
                required
                name="tablenr"
                label="Table Nr"
                :disabled="qr.manual_table_nr"
                v-model="qr.table_nr"
              >
                <div v-if="mode === 'bulk' && !bulkFields.includes('table_nr')" slot="appendLabel" class="bulkEdit" @click="enableField('table_nr', $event)">Edit</div>
              </base-input>
            </div>

            <div class="col-6">
              <base-input label="Active">
                <div v-if="mode === 'bulk' && !bulkFields.includes('active')" slot="appendLabel" class="bulkEdit" @click="enableField('active', $event)">Edit</div>
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.active"></base-switch>
              </base-input>
            </div>

            <div class="col-6">
              <base-input label="Manual table nr">
                <div v-if="mode === 'bulk' && !bulkFields.includes('manual_table_nr')" slot="appendLabel" class="bulkEdit" @click="enableField('manual_table_nr', $event)">Edit</div>
                <InfoIcon slot="appendLabel">Guests will be asked to manually enter their table number when ordering</InfoIcon>
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.manual_table_nr"></base-switch>
              </base-input>
            </div>

            <div class="col-6">
              <base-input label="Redirect disabled">
                <div v-if="mode === 'bulk' && !bulkFields.includes('redirect_disabled')" slot="appendLabel" class="bulkEdit" @click="enableField('redirect_disabled', $event)">Edit</div>
                <InfoIcon slot="appendLabel">Disable redirecting after a scan</InfoIcon>
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.redirect_disabled"></base-switch>
              </base-input>
            </div>

<!--            <div class="col-lg-auto">-->
<!--              <base-input label="Takeaway">-->
<!--                <div v-if="mode === 'bulk' && !bulkFields.includes('takeaway')" slot="appendLabel" class="bulkEdit" @click="enableField('takeaway', $event)">Edit</div>-->
<!--                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.takeaway"></base-switch>-->
<!--              </base-input>-->
<!--            </div>-->

            <div class="col-lg-auto">
              <base-input label="To Go Ordering">
                <div v-if="mode === 'bulk' && !bulkFields.includes('takeaway_onsite')" slot="appendLabel" class="bulkEdit" @click="enableField('takeaway_onsite', $event)">Edit</div>
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.takeaway_onsite"></base-switch>
              </base-input>
            </div>

<!--            <div class="col-lg-auto">-->
<!--              <base-input label="Delivery">-->
<!--                <div v-if="mode === 'bulk' && !bulkFields.includes('delivery')" slot="appendLabel" class="bulkEdit" @click="enableField('delivery', $event)">Edit</div>-->
<!--                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="qr.delivery"></base-switch>-->
<!--              </base-input>-->
<!--            </div>-->

            <div class="col-lg-6">
              <base-input
                type="text"
                disabled
                name="area"
                label="Area"
              >
                <div v-if="mode === 'bulk' && !bulkFields.includes('color')" slot="appendLabel" class="bulkEdit" @click="enableField('color', $event)">Edit</div>
                <el-select
                  v-model="qr.color"
                  placeholder="Choose area color"
                  class="select-danger"
                >
                  <el-option
                    v-for="area in areas"
                    :key="area.value"
                    :label="area.label"
                    :value="area.value">
                  </el-option>
                </el-select>
              </base-input>
            </div>

            <div class="col-lg-6">
              <base-input
                type="number"
                name="timeout"
                label="Timeout"
                v-model="qr.timeout">
              </base-input>
            </div>

          </div>

        </div>

      </form>
    </div>
    <template slot="footer">
      <base-button type="secondary" @click="onClose">{{changesMade ? 'Cancel' : 'Close' }}</base-button>
      <base-button type="primary" @click="onsubmit">{{mode === 'bulk' ? `Update ${bulkItems.length} QRs` : 'Save' }}</base-button>
    </template>
  </modal>
</template>

<script>
import swal from "sweetalert2";
import diffChecker from "@/mixins/diffChecker";

export default {
  mixins: [diffChecker],
  name: "EditQRModal",
  props: ["qrData", "show", "mode", "bulkItems"],
  data() {
    return {
      watchQRChange: true,
      changesMade: false,
      dropdownVisible: false,
      qrTemplate: {
        "code": null,
        "table_nr": null,
        "manual_table_nr": false,
        "active": true,
        "color": 'red',
        "takeaway": false,
        "takeaway_onsite": false,
        "delivery": false,
        "timeout": 15
      },
      qr: undefined,
      bulkFields: [],
      oldQR: undefined,
    };
  },
  computed: {
    areas() {return this.$store.state.areaColors}
  },
  mounted() {
    // if(this.mode === 'bulk') {
    //   this.qr = {};
    //   return;
    // }
    let self = this;
    this.qr = JSON.parse(JSON.stringify(this.qrTemplate));

    this.watchQRChange = false;
    if(this.qrData){
      this.qr = JSON.parse(JSON.stringify(this.qrData))
    }
    this.$nextTick(function () {
      this.watchQRChange = true;
    })

    this.oldQR = JSON.parse(JSON.stringify(this.qr));


  },
  watch: {
    qrData: function(newVal){
      this.watchQRChange = false;
      this.qr = JSON.parse(JSON.stringify(newVal));
      this.$nextTick(function () {
        this.watchQRChange = true;
      })
    },
    qr: {
      handler(newVal){
        if(this.watchQRChange){
          this.changesMade = true;
        }
      },
      deep: true
    }
  },
  methods: {
    onsubmit() {
      let self = this;

      if(this.mode === 'new'){
        //New obj has been saved, do POST QR call
        this.$store.dispatch('createQR', this.qr)
          .then(function(res){
            alert('created QR successfully');
            self.onClose()
          })
          .catch(function(err) {
            alert(err)
          })
      }else{
        if(this.mode === 'bulk'){
          let values = self.qr;
          Object.keys(values).forEach(key => {
            //Delete keys that aren't in the bulkFields to prevent accidental overwrites
            if(!self.bulkFields.includes(key)) delete values[key];
          })

          let timeout = 0;
          let delay = 100;

          self.bulkItems.forEach((qr, index) => {
            setTimeout(() => {
              // let qrObj = qr;
              Object.keys(values).forEach(key => {
                qr[key] = values[key];
              })


              this.$store.dispatch('updateQR', qr)
                  .then(function(){
                    if(index + 1 === self.bulkItems.length){
                      //this was the last server call
                      //TODO if a server call 100ms earlier failed or is still pending this would already close it, make sure every call is succesful before closing
                      self.onClose();
                    }
                  })
                  .catch(function(err) {
                    alert(err)
                  })
            }, timeout + index * delay);


          })

        }else{
          //A QR was edited, do a PUT call for him/her
          this.$store.dispatch('updateQR', this.qr)
              .then(function(){
                self.onClose()
              })
              .catch(function(err) {
                alert(err)
              })
        }

      }
    },
    onClose() {
      this.changesMade = false;
      this.qr = JSON.parse(JSON.stringify(this.qrTemplate));
      this.$emit('modalClose');
    },
    enableField(field, event){
      event.preventDefault(); //Disable clicking behind the div

      this.qr[field] = null;
      this.bulkFields.push(field);

      console.log(event);
    }
  },
}
</script>

<style scoped>
  .bulkEdit{
    background: #cecece;
    color: #cecece;
    opacity: 0.8;
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    width: calc(100% + 20px);
    height: calc(100% + 20px);
    margin: -10px;
    top: 0;
    left: 0;
    z-index: 2;
    cursor: pointer;
    transition: color .2s ease;
    border-radius: 6px;
  }

  .bulkEdit:hover{
    color: #000;
  }
</style>
