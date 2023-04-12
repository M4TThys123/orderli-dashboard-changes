<template>
  <div class="card p-4">
    <div v-if="currentRestaurant && restaurant">

      <form @submit.prevent="">

        <div class="row">

          <div class="col-auto">
            <base-input
              label="POS System"
            >
              <el-select
                v-model="restaurant.pos_integration"
                class="select-danger"
              >
                <el-option
                  v-for="method in [{value: null, label: 'None'},{value: 'lightspeed', label: 'Lightspeed L Series'},{value: 'lightspeed_k', label: 'Lightspeed K Series'},{value: 'untill', label: 'unTill'},{value: 'vectron', label: 'Vectron'},{value: 'mpluskassa', label: 'mPlusKASSA'},{value: 'micas', label: 'Micas'},{value: 'povis', label: 'Povis'},{value: 'matrix', label: 'Matrix'},{value: 'webhook', label: 'Webhook API'}]"
                  :key="method.value"
                  :label="method.label"
                  :value="method.value">
                </el-option>
              </el-select>
            </base-input>
          </div>
        </div>

        <div class="row" v-if="restaurant.pos_integration === 'lightspeed'">
          <div class="col-12">
            <base-button type="secondary" @click="connectPos(false)">
              <span><img src="img/icons/pos/lightspeed-logo-icon-only.svg"
                         style="width: 16px; height: auto; margin-right: 1rem" alt="Lightspeed logo"></span>
              <span>Connect Lightspeed L Series</span>
            </base-button>
            <!--            <p class="pt-1">Or (re)connect with Lightspeed on the <a href="https://web.orderli.com/admin">old admin panel</a></p>-->
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'lightspeed_k'">
          <div class="col-12">
            <base-button type="secondary" @click="connectPos(false)">
              <span><img src="img/icons/pos/lightspeed-logo-icon-only.svg"
                         style="width: 16px; height: auto; margin-right: 1rem" alt="Lightspeed logo"></span>
              <span>Connect Lightspeed K Series</span>
            </base-button>
            <el-select
              v-if="restaurant.pos_username && lightspeed_businesses"
              v-model="restaurant.lightspeed_k_business_id"
              placeholder="Choose Lightspeed K business"
              class="select-danger"
            >
              <el-option-group
                v-for="account in lightspeed_businesses"
                :key="account.id"
                :label="account.name">
                <el-option
                  v-for="profile in account.businessLocations"
                  :key="profile.id"
                  :label="profile.name + ' (' + profile.id + ')'"
                  :value="profile.id">
                </el-option>
              </el-option-group>
              <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save
              </base-button>
            </el-select>
            <!--            <p class="pt-1">Or (re)connect with Lightspeed on the <a href="https://web.orderli.com/admin">old admin panel</a></p>-->
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'untill'">
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_wsdl"
              disabled
              type="text"
              label="WSDL"
              placeholder="http://111.111.111.111:3063/wsdl/ITPAPIPOS"
              v-model="restaurant.pos_wsdl"
            >
            </base-input>
          </div>
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_endpoint"
              disabled
              type="text"
              label="Endpoint"
              placeholder="http://111.111.111.111:3063/soap/ITPAPIPOS"
              v-model="restaurant.pos_endpoint"
            >
            </base-input>
          </div>
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_username"
              disabled
              type="text"
              label="Username"
              placeholder="TPAPI-Orderli"
              v-model="restaurant.pos_username"
            >
            </base-input>
          </div>
          <div class="col-12">
            <div class="row">
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Host"
                  placeholder="testapi.untill.com:3063"
                  :value="restaurant.pos_host"
                  v-model="posSettings.host"
                >
                  <small slot="prepend" class="font-weight-bold">http://</small>
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Username"
                  placeholder="Orderli"
                  v-model="posSettings.username"
                >
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Password"
                  placeholder="aS325hFd0!sf569793hKasf@t3"
                  v-model="posSettings.password"
                >
                </base-input>
              </div>
            </div>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'vectron'">
          <div class="col col-lg-6">
            <base-input
              v-if="restaurant.pos_endpoint"
              disabled
              type="text"
              label="Endpoint"
              placeholder="http://111.111.111.111:49000/"
              v-model="restaurant.pos_endpoint"
            >
            </base-input>
          </div>
          <div class="col col-lg-6">
            <base-input
              v-if="restaurant.pos_username"
              disabled
              type="text"
              label="Username"
              placeholder="Orderli"
              v-model="restaurant.pos_username"
            >
            </base-input>
          </div>
          <div class="col-12">
            <div class="row">
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Host"
                  placeholder="111.111.111.111:3063"
                  v-model="posSettings.host"
                >
                  <small slot="prepend" class="font-weight-bold">http://</small>
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Username"
                  placeholder="Orderli"
                  v-model="posSettings.username"
                >
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Password"
                  placeholder="aS325hFd0!sf569793hKasf@t3"
                  v-model="posSettings.password"
                >
                </base-input>
              </div>
            </div>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'mpluskassa'">
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_wsdl"
              disabled
              type="text"
              label="WSDL"
              placeholder="https://api.mpluskassa.nl:12345?wsdl"
              v-model="restaurant.pos_wsdl"
            >
            </base-input>
          </div>
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_endpoint"
              disabled
              type="text"
              label="Endpoint"
              placeholder="https://api.mpluskassa.nl:12345"
              v-model="restaurant.pos_endpoint"
            >
            </base-input>
          </div>
          <div class="col col-lg-4">
            <base-input
              v-if="restaurant.pos_username"
              disabled
              type="text"
              label="Username"
              placeholder="orderli"
              v-model="restaurant.pos_username"
            >
            </base-input>
          </div>
          <div class="col-12">
            <div class="row">
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Host"
                  placeholder="111.111.111.111:3063"
                  v-model="posSettings.host"
                >
                  <small slot="prepend" class="font-weight-bold">https://</small>
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Username"
                  placeholder="Orderli"
                  v-model="posSettings.username"
                >
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Password"
                  placeholder="aS325hFd0!sf569793hKasf@t3"
                  v-model="posSettings.password"
                >
                </base-input>
              </div>
            </div>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'micas'">
          <div class="col col-lg-6">
            <base-input
              v-if="restaurant.pos_endpoint"
              disabled
              type="text"
              label="Endpoint"
              placeholder="http://111.111.111.111:9999"
              v-model="restaurant.pos_endpoint"
            >
            </base-input>
          </div>
          <div class="col-12">
            <div class="row">
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Host"
                  placeholder="111.111.111.111:3063"
                  v-model="posSettings.host"
                >
                  <small slot="prepend" class="font-weight-bold">http://</small>
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Username"
                  placeholder="Orderli"
                  v-model="posSettings.username"
                >
                </base-input>
              </div>
              <div class="col col-lg-4">
                <base-input
                  type="text"
                  label="Password"
                  placeholder="aS325hFd0!sf569793hKasf@t3"
                  v-model="posSettings.password"
                >
                </base-input>
              </div>
            </div>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'povis'">
          <div class="col col-lg-6">
            <base-input
              type="text"
              label="Povis POS ID"
              placeholder="12345"
              v-model="posSettings.povis_pos_id"
            >
            </base-input>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'matrix'">
          <div class="col col-lg-6">
            <base-input
              type="text"
              label="Matrix host"
              placeholder="192.168.1.1:1234"
              v-model="posSettings.host"
            >
            </base-input>
          </div>
        </div>
        <div class="row" v-else-if="restaurant.pos_integration === 'webhook'">
          <div class="col-12">
            <div class="row">
              <div class="col col-lg-6">
                <base-input
                  type="text"
                  label="Order url"
                  placeholder=""
                  v-model="webhookSettings.order_url"
                >
                  <!--                  <small slot="prepend" class="font-weight-bold">http://</small>-->
                </base-input>
              </div>
              <div class="col col-lg-6">
                <base-input
                  type="text"
                  label="Get Bill url"
                  placeholder=""
                  v-model="webhookSettings.get_bill_url"
                >
                </base-input>
              </div>
              <div class="col col-lg-6">
                <base-input
                  type="text"
                  label="Pay Table Bill url"
                  placeholder=""
                  v-model="webhookSettings.pay_table_bill_url"
                >
                </base-input>
              </div>
              <div class="col col-lg-6">
                <base-input
                  type="text"
                  label="Get Menu url"
                  placeholder=""
                  v-model="webhookSettings.get_menu_url"
                >
                </base-input>
              </div>
              <div class="col col-lg-12">
                <base-button type="primary" native-type="submit" @click="connectPos(true)" :disabled="submitting">Submit
                  POS settings
                </base-button>
              </div>
            </div>
          </div>

        </div>
        <div class="row" v-else-if="restaurant.pos_integration">
          <div class="col-12">
            <p>Contact us for more information on how to integrate with {{restaurant.pos_integration}}</p>
          </div>
        </div>

      </form>

      <!--      <base-button type="secondary" native-type="submit" @click="resetComponent" :disabled="!changesMade">Cancel changes</base-button>-->
      <base-button
        v-if="restaurant.pos_integration === 'untill' || restaurant.pos_integration === 'vectron' || restaurant.pos_integration === 'mpluskassa' || restaurant.pos_integration === 'micas' || restaurant.pos_integration === 'povis' || restaurant.pos_integration === 'matrix'"
        type="primary" native-type="submit" @click="connectPos(false)" :disabled="submitting || !posSettingsCompleted">
        Submit POS settings
      </base-button>

      <base-button v-if="!restaurant.pos_integration && currentRestaurant.pos_integration" type="primary"
                   native-type="submit" @click="removePos" :disabled="submitting">Save
      </base-button>

      <template
        v-if="(restaurant.pos_integration === 'untill' || restaurant.pos_integration === 'vectron') && restaurant.pos_endpoint">
        <hr>
        <!--        <pre>{{restaurant}}</pre>-->
        <base-button type="primary" native-type="submit" @click="getPayment(restaurant.pos_integration)"
                     :disabled="loadingPaymentTypes">Load Payment Types
        </base-button>

        <div class="col-auto" v-if="restaurant.pos_integration === 'untill' && paymentTypes">
          <base-input
            label="unTill payment type"
          >
            <el-select
              v-model="restaurant.untill_payment_method"
              class="select-danger"
            >
              <el-option
                v-for="method in paymentTypes"
                :key="method.id"
                :label="method.name"
                :value="method.id">
              </el-option>
            </el-select>
          </base-input>
          <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save</base-button>
        </div>
        <div class="col-auto" v-if="restaurant.pos_integration === 'vectron' && paymentTypes">
          <base-input
            label="Vectron payment type"
          >
            <el-select
              :value="vectronPaymentTypeChosen"
              v-on:change="updateVectronPaymentType"
              class="select-danger"
              value-key="Number"

            >
              <el-option
                v-for="method in paymentTypes"
                :key="method.Number"
                :label="method.Name"
                :value="method">
              </el-option>
            </el-select>
          </base-input>
          <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save</base-button>
        </div>
      </template>

      <template>
        <hr>
        <form @submit.prevent="">
          <div class="row">
            <div v-if="restaurant.pos_integration === 'mpluskassa'" class="col-lg-4">
              <base-input label="MplusKassa branch">
                <el-select
                  :value="mplusBranchChosen"
                  v-on:change="updateMplusBranch"
                  class="select-danger"
                  value-key="id">
                  <el-option
                    v-for="branch in mplusBranches"
                    :key="branch.id"
                    :label="'(' + branch.id + ')'  + branch.name"
                    :value="branch">
                  </el-option>
                </el-select>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration === 'mpluskassa'" class="col-lg-4">
              <base-input label="MplusKassa terminal">
                <el-select
                  :value="mplusTerminalChosen"
                  v-on:change="updateMplusTerminal"
                  class="select-danger"
                  value-key="id">
                  <el-option
                    v-for="terminal in mplusTerminals"
                    :key="terminal.id"
                    :label="terminal.name + '(Branch number: ' + terminal.branch_id + ')'"
                    :value="terminal">
                  </el-option>
                </el-select>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration === 'mpluskassa'" class="col-lg-4">
              <base-input label="Tip item">
                <InfoIcon slot="appendLabel">
                  What item to use to send tip amount to MplusKassa
                </InfoIcon>
                <el-select
                  v-model="restaurant.mplus_tip_item"
                  filterable
                  :placeholder="dropdownVisible ? 'Search products...' : 'Add products'"
                  v-on:visible-change="dropdownVisible = $event"
                  class="select-danger">
                  <el-option
                    v-for="item in allItems"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id">
                    <tippy :delay="[1000,0]" animation="fade" :duration="0" :animationFill="false" boundary="window"
                           placement="top-start" theme="no-padding">
                      <template v-slot:trigger><span>{{item.name}} ({{item.option_groups.length}})</span></template>
                    </tippy>
                  </el-option>
                </el-select>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration === 'untill'" class="col-lg-4">
              <base-input label="Error item">
                <InfoIcon slot="appendLabel">
                  Send this item to the POS when an error occurs
                </InfoIcon>
                <el-select
                  v-model="restaurant.error_item"
                  filterable
                  :placeholder="dropdownVisible ? 'Search products...' : 'Add products'"
                  v-on:visible-change="dropdownVisible = $event"
                  class="select-danger">
                  <el-option
                    v-for="item in allItems"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id">
                    <tippy :delay="[1000,0]" animation="fade" :duration="0" :animationFill="false" boundary="window"
                           placement="top-start" theme="no-padding">
                      <template v-slot:trigger><span>{{item.name}} ({{item.option_groups.length}})</span></template>
                    </tippy>
                  </el-option>
                </el-select>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration" class="col-lg-4">
              <base-input label="Delivery costs item">
                <InfoIcon slot="appendLabel">
                  What item to use to send delivery costs to the POS
                </InfoIcon>
                <el-select
                  v-model="restaurant.delivery_costs_item"
                  filterable
                  :placeholder="dropdownVisible ? 'Search products...' : 'Add products'"
                  v-on:visible-change="dropdownVisible = $event"
                  class="select-danger">
                  <el-option
                    v-for="item in allItems"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id">
                    <tippy :delay="[1000,0]" animation="fade" :duration="0" :animationFill="false" boundary="window"
                           placement="top-start" theme="no-padding">
                      <template v-slot:trigger><span>{{item.name}} ({{item.option_groups.length}})</span></template>
                    </tippy>
                  </el-option>
                </el-select>
              </base-input>
            </div>
          </div>
          <div class="row">
            <div v-if="restaurant.pos_integration === 'untill'" class="col-lg-4">
              <base-input label="unTill Member Number Input">
                <base-switch class="mr-1" on-text="On" off-text="Off"
                             v-model="restaurant.untill_client_input"></base-switch>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration === 'untill'"  class="col-lg-4">
              <base-input label="Check stock">
                <base-switch class="mr-1" on-text="On" off-text="Off"
                             v-model="restaurant.untill_check_stock"></base-switch>
              </base-input>
            </div>
            <div v-if="restaurant.pos_integration === 'vectron'"  class="col-lg-4">
              <base-input label="Check if table is available">
                <base-switch class="mr-1" on-text="On" off-text="Off"
                             v-model="restaurant.vectron_check_table"></base-switch>
              </base-input>
            </div>
            <template v-if="restaurant.pos_integration">
              <div class="col-lg-4">
                <base-input label="Autoswipe">
                  <InfoIcon slot="appendLabel">
                    Send orders automatically to the connected POS. This settings stays on even after
                    closing the admin page.
                  </InfoIcon>
                  <base-switch class="mr-1" on-text="On" off-text="Off"
                               v-model="restaurant.auto_swipe"></base-switch>
                </base-input>
              </div>
              <div class="col-lg-4">
                <base-input label="Auto import">
                  <i v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('auto_import')" class="fas fa-lock" slot="prependLabel" style="font-size: 12px"></i>
                  <badge v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('auto_import')" slot="appendLabel" rounded type="secondary" class="ml-2">PRO</badge>
                  <base-switch class="mr-1" on-text="On" off-text="Off"
                     :disabled="currentAccount.subscription === null ||
                     !currentAccount.subscription.features.includes('auto_import')"
                     v-model="restaurant.auto_pos_import_enabled">
                  </base-switch>
                </base-input>
              </div>
              <div class="col-lg-4" v-if="isAdmin">
                <base-input label="Auto update items (ADMIN ONLY FOR NOW — Please consult before turning this on as it might mess up the complete menu)">
                  <i v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('auto_import')" class="fas fa-lock" slot="prependLabel" style="font-size: 12px"></i>
                  <badge v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('auto_import')" slot="appendLabel" rounded type="secondary" class="ml-2">PRO</badge>
                  <base-switch class="mr-1" on-text="On" off-text="Off"
                     :disabled="currentAccount.subscription === null ||
                     !currentAccount.subscription.features.includes('auto_import')"
                     v-model="restaurant.pos_import_auto_change">
                  </base-switch>
                </base-input>
              </div>
            </template>
            <div class="col-lg-12" v-if="restaurant.pos_import_auto_change === true">
              <h6 class="heading-small text-muted mb-2 d-inline-block">Auto update specific data</h6>
              <div class="row">
                <div class="col-lg-4" v-for="data in getAutoUpdateData">
                  <base-checkbox v-model="data.enabled">
                    {{ data['name'] }}
                  </base-checkbox>
                </div>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-lg-12">
              <base-button type="primary" native-type="submit" @click="submit" :disabled="!changesMade">
                Save changes
              </base-button>
            </div>
          </div>
        </form>
      </template>

      <br>
      <div v-if="(restaurant.pos_endpoint || restaurant.pos_username || restaurant.povis_pos_id) && restaurant.pos_integration">
        <hr>

        <base-button type="primary" class="mt-4" native-type="submit" @click="showPosOverviewModal = true"
                     v-if="restaurant.pos_imported_at">
          <i class="fa fa-list-ul"></i>
          Show all POS items
        </base-button>
        <base-button type="primary" class="mt-4" native-type="submit" @click="clearPosItems"
                     v-if="restaurant.pos_imported_at">
          <i class="fa fa-trash-alt"></i>
          Clear POS items
        </base-button>
        <base-button type="primary" class="mt-4" native-type="submit" @click="startPosImport" :disabled="submitting">
          <i class="fa fa-sync"></i>
          Import menu from POS
        </base-button>
<!--        <base-button type="primary" class="mt-4" native-type="submit" @click="showPosRequests">-->
<!--          <i class="fa fa-brackets-curly"></i>-->
<!--          Show request logs-->
<!--        </base-button>-->
        <div class="mt-4" v-if="posImportStatus !== null">
          <template v-if="posImportStatus.status === 'success'">
            <small style="color: #2ed92e;"><i class="fa fa-check-circle"></i>POS Import succeeded</small><br>
            <small>Succeeded at: {{formatDate(posImportStatus.last_imported_at)}}</small>
          </template>
          <template v-else-if="posImportStatus.status === 'pending'">
            <small style="color: #787878;"><i class="fa fa-clock"></i>POS Import running...</small><br>
            <small>Started: {{formatDate(posImportStatus.requested_at)}}</small>
          </template>
          <template v-else>
            <small style="color: #e00f0f;"><i class="fa fa-times-circle"></i>Last import failed</small><br>
            <small>Last import started at: {{formatDate(posImportStatus.requested_at)}}</small>
          </template>

        </div>
      </div>

      <PosItemsOverviewModal
        v-if="showPosOverviewModal && currentRestaurant && restaurant.pos_imported_at"
        :show="showPosOverviewModal" v-on:modalClose="showPosOverviewModal = false"
        :lastImport="formatDate(restaurant.pos_imported_at)"/>

    </div>

    <div v-else class="d-flex justify-content-center py-8">
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>

    <!--    Debug connection-->
    <!--    Test POS printers-->
    <!--    Request integration-->

  </div>
</template>

<script>
  import store from "@/store";
  import formatDateTime from "@/mixins/formatDateTime";
  import restaurantSettingsForm from "@/mixins/restaurantSettingsForm";
  import PosItemsOverviewModal from "@/views/Products/PosItemsOverviewModal";

  export default {
    mixins: [restaurantSettingsForm, formatDateTime],
    name: "PosSettings",
    components: {PosItemsOverviewModal},
    data() {
      return {
        dropdownVisible: false,
        submitting: false,
        posSettings: {
          host: null,
          username: null,
          password: null,
          povis_pos_id: null,
        },
        webhookSettings: {
          order_url: null,
          get_bill_url: null,
          pay_table_bill_url: null,
          get_menu_url: null,
        },
        prepend: null,
        showPosOverviewModal: false,
        lightspeed_businesses: null,
        loadingPaymentTypes: false,
        paymentTypes: null,
        vectronPaymentTypeChosen: null,
        loadingMplusBranches: false,
        mplusBranches: null,
        mplusBranchChosen: null,
        loadingMplusTerminals: false,
        mplusTerminals: null,
        mplusTerminalChosen: null,
        posImportStatus: null,
        posImportStatusChecker: null,
        autoUpdateData: [
          {
            key: "name",
            name: "Name",
            enabled: true
          },
          {
            key: "price",
            name: "Price",
            enabled: true
          },
          {
            key: "vat",
            name: "Vat",
            enabled: true
          },
          {
            key: "option_groups",
            name: "Option groups",
            enabled: true
          },
          {
            key: "translations",
            name: "Translations",
            enabled: true
          },
          {
            key: "untill_item_type",
            name: "Untill item type",
            enabled: true
          },
          {
            key: "lightspeed_k_is_modifier",
            name: "Lightspeed K is modifier",
            enabled: true
          },
          {
            key: "lightspeed_k_is_subitem",
            name: "Lightspeed K is subitem",
            enabled: true
          },
          {
            key: "image",
            name: "Images",
            enabled: true
          },
        ]
      }
    },
    computed: {
      currentAccount() {
        return this.$store.getters.currentAccount;
      },
      posSettingsCompleted() {
        return this.restaurant.pos_integration === 'povis' ? this.posSettings.povis_pos_id : this.posSettings.host && this.posSettings.username && this.posSettings.password;
      },
      getAutoUpdateData() {
        let self = this;
        if (this.restaurant.pos_import_auto_change_keys.length > 0) {
          this.autoUpdateData.forEach((updateData) => {
            updateData.enabled = self.restaurant.pos_import_auto_change_keys.includes(updateData.key);
          })
        }
        return this.autoUpdateData;
      },
      isAdmin() {
        return this.$store.getters.isAdmin;
      },
      allItems() {
        return this.$store.getters.itemsOfCurrentRestaurant;
      },
    },
    watch: {
      'restaurant.pos_integration': function (newVal) {
        switch (newVal) {
          case "untill":
            this.prepend = 'http://'
            break;
          case "vectron":
            this.prepend = 'http://'
            break;
          case "mpluskassa":
            this.prepend = 'https://'
            break;
          case "micas":
            this.prepend = 'http://'
            break;
          case "index":
            this.prepend = 'http://'
            break;
          default:
            this.prepend = null;
        }
      },
    },
    mounted() {
      let self = this;

      if (this.restaurant === undefined) {
        //if mounted hook executes before methods call was finished restaurant would still be undefined, check for that
        const watcher = store.watch(() => this.restaurant, restaurantLoaded => {
          watcher(); // stop watching
          if (restaurantLoaded) {
            if (self.restaurant.pos_integration !== null) {
              this.getPosImportStatus();
            }

            if (self.restaurant.pos_integration === 'lightspeed_k' && self.restaurant.pos_username) {
              self.$store.dispatch('getLightspeedKBusinesses')
                .then(function (res) {
                  self.lightspeed_businesses = res.data;
                })
            }

            if (self.restaurant.pos_integration === 'mpluskassa' && self.restaurant.pos_username) {
              if (self.restaurant.mplus_branch_number !== null && self.restaurant.mplus_branch_name) {
                self.mplusBranchChosen = {
                  'id': self.restaurant.mplus_branch_number,
                  'name': self.restaurant.mplus_branch_name
                }
              }

              if (self.restaurant.mplus_terminal_number !== null && self.restaurant.mplus_terminal_name) {
                self.mplusTerminalChosen = {
                  'id': self.restaurant.mplus_terminal_number,
                  'name': self.restaurant.mplus_terminal_name
                }
              }

              self.getMplusBranches()
              self.getMplusTerminals()
            }
          }
        });
      } else {
        if (self.restaurant.pos_integration !== null) {
          this.getPosImportStatus();
        }

        if (self.restaurant.pos_integration === 'lightspeed_k' && self.restaurant.pos_username) {
          self.$store.dispatch('getLightspeedKBusinesses')
            .then(function (res) {
              self.lightspeed_businesses = res.data;
            })
        }

        if (self.restaurant.pos_integration === 'mpluskassa' && self.restaurant.pos_username) {
          if (self.restaurant.mplus_branch_number !== null && self.restaurant.mplus_branch_name) {
            self.mplusBranchChosen = {
              'id': self.restaurant.mplus_branch_number,
              'name': self.restaurant.mplus_branch_name
            }
          }

          if (self.restaurant.mplus_terminal_number !== null && self.restaurant.mplus_terminal_name) {
            self.mplusTerminalChosen = {
              'id': self.restaurant.mplus_terminal_number,
              'name': self.restaurant.mplus_terminal_name
            }
          }

          self.getMplusBranches()
          self.getMplusTerminals()
        }
      }
    },
    beforeDestroy() {
      if (this.posImportStatusChecker) {
        this.stopPosImportStatusCheck();
      }
    },
    methods: {
      getPosImportStatus() {
        let self = this;
        this.$store.dispatch('getPosImportStatus')
          .then((res) => {
            self.posImportStatus = res.data;

            if (self.posImportStatus.status === 'success' && self.posImportStatusChecker !== null) {
              self.stopPosImportStatusCheck();
            } else if (self.posImportStatus.status === 'pending' && self.posImportStatusChecker === null) {
              self.startPosImportStatusCheck();
            }
          })
          .catch((err) => {

          })
      },
      startPosImport() {
        let self = this;
        this.submitting = true;
        this.$store.dispatch('importPosMenu')
          .then(function () {
            self.submitting = false;
            alert('POS import started.');
            self.startPosImportStatusCheck();
          })
          .catch(function (err) {
            self.submitting = false;
            if(err.response && err.response.data.message) alert(err.response.data.message)
            else(alert(err))
          })
      },
      startPosImportStatusCheck() {
        let self = this;
        this.posImportStatusChecker = setInterval(() => {
          if (self.restaurant.pos_integration !== null) {
            self.getPosImportStatus();
          } else {
            self.stopPosImportStatusCheck();
          }
        }, 5000)
      },
      stopPosImportStatusCheck() {
        clearInterval(this.posImportStatusChecker);
      },
      clearPosItems() {
        this.$store.dispatch('clearPosItems')
          .then(function () {
            alert('All POS items have been succesfully deleted.');
          })
          .catch(function (err) {
            alert(err)
          })
      },
      removePos() {
        let self = this;
        this.submitting = true;
        this.$store.dispatch('setPosIntegration', {pos: null})
          .then(function () {
            self.submitting = false;
            alert('POS integration turned off.');
          })
          .catch(function (err) {
            self.submitting = false;
            alert(err)
          })
      },
      connectPos(isWebhook) {
        let posSettings = null;
        if (isWebhook) {
          posSettings = {
            webhook_order_url: this.webhookSettings.order_url,
            webhook_get_bill_url: this.webhookSettings.get_bill_url,
            webhook_pay_table_bill_url: this.webhookSettings.pay_table_bill_url,
            webhook_get_menu_url: this.webhookSettings.get_menu_url,
          }
        } else {
          posSettings = {
            pos: this.restaurant.pos_integration,
            host: this.prepend + this.posSettings.host,
            username: this.posSettings.username,
            password: this.posSettings.password,
            povis_pos_id: this.posSettings.povis_pos_id,
          }
        }

        let self = this;
        this.submitting = true;

        this.$store.dispatch('setPosIntegration', posSettings)
          .then(function () {
            self.submitting = false;
            self.posSettings = {
              host: null,
              username: null,
              password: null,
            }
            alert('POS details saved, please test it by refreshing the page and then clicking \' import menu from POS\'. When reloading after a few minutes there is a \'last imported at\' time the connection was successful.');
          })
          .catch(function (err) {
            self.submitting = false;
            alert(err)
          })

      },
      getPayment(pos) {
        let self = this;
        this.loadingPaymentTypes = true;
        this.$store.dispatch('getPayments', pos)
          .then(res => {
            console.log(res);
            self.paymentTypes = res.payments || res.Payments || res;
            self.loadingPaymentTypes = false;
          })
          .catch(err => {
            alert(err);
            self.loadingPaymentTypes = false
          })
      },
      updateVectronPaymentType(value) {
        this.vectronPaymentTypeChosen = value;
        this.restaurant.vectron_payment_type_id = value.Number;
        this.restaurant.vectron_payment_type_name = value.Name;
      },
      getMplusBranches() {
        let self = this;
        this.$store.dispatch('getMplusBranches')
          .then(res => {
            self.mplusBranches = res.data;
          })
          .catch(err => {

          })
      },
      updateMplusBranch(value) {
        console.log(value);
        this.mplusBranchChosen = value;
        this.restaurant.mplus_branch_number = value.id;
        this.restaurant.mplus_branch_name = value.name;
      },
      getMplusTerminals() {
        let self = this;
        this.$store.dispatch('getMplusTerminals')
          .then(res => {
            self.mplusTerminals = res.data;
          })
          .catch(err => {

          })
      },
      updateMplusTerminal(value) {
        console.log(value);
        this.mplusTerminalChosen = value;
        this.restaurant.mplus_terminal_number = value.id;
        this.restaurant.mplus_terminal_name = value.name;
      },
      submit() {
        let pos_import_auto_change_keys = []
        this.getAutoUpdateData.forEach((updateData) => {
          if (updateData.enabled) {
            pos_import_auto_change_keys.push(updateData.key)
          }
        })
        this.restaurant.pos_import_auto_change_keys = pos_import_auto_change_keys;
        this.onSubmit();
      }
    }
  }
</script>

<style scoped>

</style>
