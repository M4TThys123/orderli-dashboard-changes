<template>
  <div class="device-frame rounded shadow position-relative">
    <iframe ref="previewIframe" :src="`${serverUrl}${currentRestaurant.database_name}?menu_id=${selectedMenu}&menuEditor=true`" @load="setupPreview()" width="100%" height="100%" class="device-screen rounded" />
    <el-select
      v-model="previewDay"
      filterable
      placeholder="Day"
      class="select-danger position-absolute top-0 left-0 shadow-sm"
      style="transform: translate(0, -50%); width:130px"
    >
      <el-option
        v-for="day in weekdays"
        :key="day.value"
        :label="day.label"
        :value="day.value">
      </el-option>
    </el-select>
    <input
      :value="formatToTimestring(previewTime)"
      v-on:blur="formatToSecondsTimeslot($event)"
      type="time"
      class="form-control position-absolute top-0 left-0 shadow-sm"
      style="transform: translate(140%, -50%); width: unset;"
    />
    <el-tooltip content="Reload preview" placement="top">
      <span @click="reloadPreview" class="badge badge-lg badge-circle badge-floating border-dark position-absolute top-0 right-0 shadow-sm" style="transform: translate(-225%, -50%); cursor: pointer; background-color: #e9ecef">
        <i class="fas fa-undo"></i>
      </span>
    </el-tooltip>
    <el-tooltip effect="light" placement="bottom">
      <div slot="content">
        <h3 class="text-center text-uppercase" style="color: #525f7f; letter-spacing: 1px; font-weight: 800;">Preview on phone</h3>
        <VueQRCodeComponent :text="`${serverUrl}${currentRestaurant.database_name}?menu_id=${selectedMenu}`" :size="200" color="#525f7f" bg-color="#fff"></VueQRCodeComponent>

      </div>
      <span class="badge badge-lg badge-circle badge-floating border-dark position-absolute top-0 right-0 shadow-sm" style="transform: translate(-100%, -50%); cursor: pointer; background-color: #e9ecef">
        <i class="fas fa-qrcode"></i>
      </span>
    </el-tooltip>
    <el-tooltip content="Open in new window" placement="top">
      <a :href="`${serverUrl}${currentRestaurant.database_name}?menu_id=${selectedMenu}`" target="_blank" class="badge badge-lg badge-circle badge-floating border-dark position-absolute top-0 right-0 shadow-sm" style="transform: translate(25%, -50%); cursor: pointer; background-color: #e9ecef">
        <i class="fas fa-external-link-alt"></i>
      </a>
    </el-tooltip>
<!--    <div class="absolute-center">-->
<!--      <button @click="reloadPreview" class="btn btn-primary shadow-lg position-absolute" style="bottom:-15px">Reload preview</button>-->
<!--    </div>-->
  </div>
</template>

<script>
import VueQRCodeComponent from 'vue-qrcode-component'

export default {
  name: "MenuPreview",
  props: ["thuis", "selectedMenu", "currentMenu", "restaurant"],
  data() {
    return {
      previewDay: 0, //TODO save and load from localstorage
      previewTime: 43200, //12:00 //TODO save and load from localstorage
      weekdays: [
        {
          value: 0,
          label: "Monday",
        },
        {
          value: 1,
          label: "Tuesday",
        },
        {
          value: 2,
          label: "Wednesday",
        },
        {
          value: 3,
          label: "Thursday",
        },
        {
          value: 4,
          label: "Friday",
        },
        {
          value: 5,
          label: "Saturday",
        },
        {
          value: 6,
          label: "Sunday",
        },
      ],
    };
  },
  components: {
    VueQRCodeComponent,
  },
  computed: {
    currentRestaurant() {return this.$store.getters.currentRestaurant},
    serverUrl(){
      if(this.thuis) return 'https://thuis.orderli.com/'
      else return process.env.VUE_APP_SERVER_URL || 'https://web.orderli.com/'
    },
    activeMenu() {
      let self = this;
      let menu = JSON.parse(JSON.stringify(this.$store.state.menu.menu))
      // menu = { ...this.$store.state.menu.menu };
      menu.cards = menu.cards.filter(card => this.currentMenu?.cards.includes(card.id) && (card.card_times === null || card.card_times.length === 0 || card.card_times.some(time => (time.day === this.previewDay || time.day === -1) && this.previewTime >= time.start && (this.previewTime < time.end || time.end === 0)))).sort((a,b) => this.currentMenu.cards.indexOf(a.id) - this.currentMenu.cards.indexOf(b.id))
      menu.categories = menu.categories.filter(category =>
        category.category_times === null || category.category_times.length === 0 ||
        category.category_times.some(time => (time.day === this.previewDay || time.day === -1) &&
          this.previewTime >= time.start &&
          (this.previewTime < time.end || time.end === 0)))

      menu.option_groups.forEach(optionGroup => {
        optionGroup.items = optionGroup.items.map(item => {
          let overrides = [];
          optionGroup.parentItem.forEach(parent => {
            parent.option_overrides.forEach(override => {
              if(override.option_group_id === optionGroup.id && override.option_item_id === item) overrides.push(override)
            })
          })
          return ({'id': item, 'item_id': item, option_overrides: overrides})
        })
      })

      return menu;
    },
    initResponse() {
      if(!this.currentMenu || !this.restaurant) return {}
      let obj = {...this.restaurant}
      obj.authenticated = false; //wordt nog niet gebruikt
      obj.authorized_restaurant = false; //wordt nog niet gebruikt
      obj.borrelblok = null; //wordt niet meer gebruikt
      obj.delivery = this.currentMenu.thuis && this.restaurant.delivery_enabled;
      obj.takeaway = this.currentMenu.thuis && this.restaurant.takeaway_enabled
      obj.takeaway_onsite = this.currentMenu.takeaway_onsite;
      obj.has_coupons = false //TODO nog niet makkelijk te checken
      obj.has_gifty = !!this.restaurant.gifty_api_key;
      obj.idempotency_token = "testtesttesttest"
      obj.manual_table_nr = false;
      obj.payment_integration = !!this.restaurant.psp_integration;
      obj.payment_optional = this.currentMenu.payment_optional;
      obj.payment_required = this.currentMenu.payment_required;
      obj.background_color = this.currentMenu.background_color;
      obj.background_image = this.currentMenu.background_image;
      obj.table_nr = 'test';
      obj.text_alert = this.currentMenu.text_alert;
      obj.untill_price_level_id = this.currentMenu.untill_price_level_id;

      return obj;
    }
  },
  mounted() {
    let d = new Date();
    this.previewDay = (d.getDay() +6) % 7 //Convert current day of the week to Monday = 0, Tues = 1
    this.previewTime = d.getHours()*3600 + d.getMinutes()*60 + d.getSeconds();
  },
  watch: {
    activeMenu: {
      handler(newVal){
        this.updatePreview();
      },
      deep: true
    },
    initResponse: {
      handler(newVal){
        console.log('init geupdatet');
        this.updateInit();
      },
      deep: true
    },
    currentMenu: {
      handler(newVal){
        console.log('menu geupdatet');
        this.updateInit();
      },
      deep: true
    },
    restaurant: {
      handler(newVal){
        console.log('restaurant geupdatet');
        this.updateInit();
      },
      deep: true
    },

  },
  methods: {
    reloadPreview(){
      this.$refs.previewIframe.src = this.$refs.previewIframe.src; //TODO remove, deprecated
    },
    setupPreview(){
      this.updateInit();
      this.updatePreview();
    },
    updatePreview(){
      console.log('refresh menu');
      this.$refs.previewIframe.contentWindow.postMessage({call:'sendValue', value: this.activeMenu}, '*');
    },
    updateInit(){
      this.$refs.previewIframe.contentWindow.postMessage({call:'updateInit', value: this.initResponse}, '*');
    },
    updateCardPreview(id){
      this.updatePreview();
      this.selectCard(id)
    },
    selectCard(id){
      this.$refs.previewIframe.contentWindow.postMessage({call:'selectCard', value: id}, '*');
    },
    selectCategory(obj){
      this.$refs.previewIframe.contentWindow.postMessage({call:'selectCategory', value: obj}, '*');
    },
    selectItem(obj){
      this.$refs.previewIframe.contentWindow.postMessage({call:'selectItem', value: obj}, '*');
    },
    formatToTimestring(secondsfrommidnight) {
      if(isNaN(secondsfrommidnight)) {
        return null;
      }
      if (secondsfrommidnight === undefined) return null
      if(secondsfrommidnight >= 86400){secondsfrommidnight = 0}
      let h = Math.floor(secondsfrommidnight / 3600);
      let m = Math.floor(secondsfrommidnight % 3600 / 60);
      //TODO IMPORTANT this function is triggered on every change in the form / card item
      return ('0' + h).slice(-2) + ":" + ('0' + m).slice(-2);
    },
    formatToSecondsTimeslot(e){
      let timeInput = e.target.value.split(':');

      let seconds = (+timeInput[0]) * 60 * 60 + (+timeInput[1]) * 60;
      this.previewTime = seconds;
    },
  }
}
</script>

<style scoped>
.device-frame {
  display: block;
  margin: 0 auto;
  width: 100%;
  margin: 0 15px;
  height: 80vh;
  text-align: center;
  /*border-radius: 1em;*/
  /*box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);*/
}

.device-screen {
  border: none;
  border-radius: 1em;
  overflow: hidden;
  background: white;
}

.absolute-center {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.fas{
  color: #525f7f;
  margin-left: 1px;
  margin-top: 1px;
  font-size: 0.8rem;
}

</style>
