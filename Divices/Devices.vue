<template>
  <div class="content">
    <div class="container-fluid mt-4">
      <div>
        <div class="card">
          <div class="border-0 card-header">
            <h3 class="mb-0 pr-4 d-inline-block">Devices</h3>
            <base-button
              @click="addTablet"
              class="px-3"
              type="primary">
              <i class="fa fa-plus pr-2" style="font-size: .8rem;"></i>
              Link new tablet
              <i class="fas fa-lock" style="margin-left: 5px; font-size: 12px"></i>
            </base-button>
<!--            <base-button-->
<!--              @click="currentAccount.subscription === null ||-->
<!--              !currentAccount.subscription.features.includes('unlimited_tablets') ? showUpgradeModal('unlimited_tablets') : addTablet"-->
<!--              class="px-3"-->
<!--              type="primary">-->
<!--              <i class="fa fa-plus pr-2" style="font-size: .8rem;"></i>-->
<!--              Link new tablet-->
<!--              <i class="fas fa-lock" style="margin-left: 5px;" v-if="currentAccount.subscription === null ||-->
<!--              !currentAccount.subscription.features.includes('unlimited_tablets')"></i>-->
<!--            </base-button>-->
          </div>
          <el-table
            :data="allTablets">
            <el-table-column
              prop="id"
              label="id"
              sortable
              width="80">
            </el-table-column>
            <el-table-column
              prop="username"
              label="Name"
              sortable
              min-width="120">
            </el-table-column>
            <el-table-column
              prop="disabled"
              label="Disabled"
              sortable
              width="100">
            </el-table-column>
            <el-table-column
              prop="app_version"
              label="App version"
              sortable
              width="130">
              <template v-slot="{row}">
                <span :class="{'outdated' : row.app_version >= latestAppVersion }">{{row.app_version}}</span>
              </template>
            </el-table-column>
            <el-table-column label="Last active at"
                             prop="last_active_at"
                             width="140px"
                             sortable>
              <template v-slot="{row}">
                <!--                TODO idee: deze waarde wordt geüpdatet bij elke request die de tablet maakt, groen pulserend live bolletje, oranje bolletje, rood bolletje-->
                <span>{{formatDate(row.last_active_at)}}</span>
              </template>
            </el-table-column>
<!--            <el-table-column-->
<!--              label="Link to device"-->
<!--              width="135">-->
<!--              <template v-slot="{row}">-->
<!--                <button class="btn btn-primary" @click="relinkTablet(row)">Relink-->
<!--                  <InfoIcon icon="fas fa-exclamation-triangle" color="white">Clicking relink will automatically logout-->
<!--                    this tablet from Orderli.-->
<!--                  </InfoIcon>-->
<!--                </button>-->
<!--              </template>-->
<!--            </el-table-column>-->

            <el-table-column width="90px">
              <template v-slot="{row}">
                <el-dropdown trigger="click" class="dropdown">
                    <span @click.stop class="btn btn-sm btn-icon-only text-light">
                      <i class="fas fa-ellipsis-v mt-2"></i>
                    </span>
                  <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                    <el-dropdown-item class="dropdown-item" @click.native="showPrinterStatus(row.id)">
                      Show printer status
                    </el-dropdown-item>
                    <el-dropdown-item class="dropdown-item" @click.native="editTablet(row.id)">
                      Edit tablet
                    </el-dropdown-item>
                    <el-dropdown-item class="dropdown-item" @click.native="relinkTablet(row)">
                      Relink
                      <InfoIcon icon="fas fa-exclamation-triangle">Clicking relink will automatically logout this tablet from Orderli.</InfoIcon>
                    </el-dropdown-item>
                    <el-dropdown-item class="dropdown-item" @click.native="deleteTablet(row.id)">
                      Remove tablet
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </el-dropdown>

                <EditDeviceModal v-if="row.id === showEditDeviceId" v-bind:device-data="row"
                                 v-bind:show="row.id === showEditDeviceId && showEditDeviceModal"
                                 v-on:modalClose="closeEditDeviceModal"/>

                <PrinterStatusModal v-if="row.id === showPrinterStatusDeviceId"
                                    v-bind:printer-statuses="row.printer_statuses"
                                    v-bind:show="row.id === showPrinterStatusDeviceId && showPrinterStatusModal"
                                    v-on:modalClose="closePrinterStatusModal"/>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
    </div>

    <modal :show.sync="showQR">
      <h6 slot="header" class="modal-title" id="modal-title-default">Scan the code with the Orderli app on your
        tablet</h6>
      <div slot="body" class="mx-auto">

      </div>
      <!--      TODO auto refresh of token after 10 min-->
      <p style="text-align: center;">This code is valid for the <span style="font-weight: bold;">next 10 minutes</span>
        <br>and only works for <span style="font-weight: bold;">one tablet</span></p>
      <div class="d-flex align-item-center justify-content-center">
        <div class="d-inline-block">
          <VueQRCodeComponent v-if="tokenCreated && !QRExpired" :text="QRText" :size="250" color="#172b4d"
                              bg-color="#fff" style="pointer-events: none"></VueQRCodeComponent>
          <div v-else-if="QRExpired">
            The code has expired. Please reload the page and click relink tablet on the devices screen to connect the
            tablet.
            <base-button type="primary" @click="window.location.reload()">Reload</base-button>
          </div>
          <div v-else class="d-flex justify-content-center py-8">
            <div class="spinner-border text-primary" role="status">
              <span class="sr-only">Loading...</span>
            </div>
          </div>
        </div>
      </div>
      <template slot="footer">
        <base-button type="link" class="ml-auto" @click="showQR = false">Close</base-button>
      </template>
    </modal>
    <UpgradeModal :show="showModal" :feature="showUpgradeModalFeature" v-on:modalClose="showModal = false"/>
  </div>
</template>

<script>
  import swal from "sweetalert2";
  import EditDeviceModal from "./EditDeviceModal";
  import PrinterStatusModal from "./PrinterStatusModal";
  import VueQRCodeComponent from 'vue-qrcode-component'
  import {Dropdown, DropdownItem, DropdownMenu, Table, TableColumn} from "element-ui";
  import UpgradeModal from "../../components/UpgradeModal";

  export default {
    name: "Devices",
    data() {
      return {
        QRText: '',
        showQR: false,
        tokenCreated: false,
        QRCreationDate: null,
        QRCodeInterval: null,
        QRExpired: false,
        showEditDeviceId: null,
        showEditDeviceModal: false,
        showPrinterStatusDeviceId: null,
        showPrinterStatusModal: false,
        showModal: false,
        showUpgradeModalFeature: null,
      }
    },
    components: {
      UpgradeModal,
      EditDeviceModal,
      PrinterStatusModal,
      VueQRCodeComponent,
      [Table.name]: Table,
      [TableColumn.name]: TableColumn,
      [Dropdown.name]: Dropdown,
      [DropdownItem.name]: DropdownItem,
      [DropdownMenu.name]: DropdownMenu,
    },
    mounted() {
      if (this.allTablets.length < 1) {
        this.$store.dispatch('getAllTablets')
      }
    },
    beforeDestroy() {
      clearInterval(this.QRCodeInterval)
    },
    computed: {
      currentAccount() {
        return this.$store.getters.currentAccount;
      },
      allTablets() {
        return this.$store.getters.allTabletsOfCurrentRestaurant
      },
      latestAppVersion() {
        return this.$store.state.devices.latestAppVersion
      },
      allAccounts() {
        let accounts = this.$store.state.accounts;
        let newAccountsObj = {};
        accounts.forEach(function (acc) {
          let name = acc.name ? acc.name : 'NO NAME';
          newAccountsObj[acc.id] = acc.id + ' ' + name;
        })
        return newAccountsObj;
      },
    },
    methods: {
      formatDate(dateString) {
        if (!dateString) return 'never'
        let date = new Date(dateString);
        let dayOfMonth = date.getDate();
        let month = date.getMonth() + 1;
        let year = date.getFullYear();
        let hour = date.getHours();
        let minutes = date.getMinutes();
        let diffMs = new Date() - date;
        let diffSec = Math.round(diffMs / 1000);
        let diffMin = diffSec / 60;
        let diffHour = diffMin / 60;

        // formatting
        // year = year.toString().slice(-2);
        month = month < 10 ? '0' + month : month;
        dayOfMonth = dayOfMonth < 10 ? '0' + dayOfMonth : dayOfMonth;
        hour = hour < 10 ? '0' + hour : hour;
        minutes = minutes < 10 ? '0' + minutes : minutes;

        if (diffSec < 1) {
          return 'Right now';
        } else if (diffMin < 1) {
          return `${diffSec} sec. ago`
        } else if (diffHour < 1) {
          return `${diffMin} min. ago`
        } else {
          return `${dayOfMonth}-${month}-${year} ${hour}:${minutes}`
        }
      },
      showUpgradeModal(feature) {
        this.showUpgradeModalFeature = feature;
        this.showModal = true;
      },
      addTablet() {
        let self = this;
        swal.fire({
          title: "Link new tablet",
          input: "text",
          html: '<label for="my-input">Please give a name for this tablet</label>',
          showCancelButton: true,
          reverseButtons: true,
          animation: "slide-from-top",
          inputValidator: (value) => {
            if (!value) {
              return 'Name cannot be empty!'
            }
          }
        }).then((result) => {
          if (result.isConfirmed) {
            self.tokenCreated = false;
            self.QRExpired = false;
            this.showQR = true;
            self.$store.dispatch('addTablet', result.value)
              .then(function (res) {
                self.QRText = res.data.token;
                self.tokenCreated = true;
                self.QRCreationDate = new Date();
                self.QRCodeInterval = setInterval(function () {
                  if (new Date() - self.QRCreationDate > 600000) {
                    //QR code has expired
                    self.QRExpired = true;
                  }

                }, 5000);
              })
              .catch(function (err) {
                alert('something went wrong')
                console.log(err)
              })
          } else {
            //Swal input was closed with cancel
          }
        })
      },
      relinkTablet(row) {
        let self = this;
        swal.fire({
          title: 'Are you sure?',
          text: "If this tablet is still logged in it will be logged out automatically!",
          icon: 'warning',
          showCancelButton: true,
          reverseButtons: true,
          confirmButtonColor: '#d62828',
          cancelButtonColor: '#929292',
          confirmButtonText: `Yes, relink ${row.username}!`
        }).then((result) => {
          if (result.isConfirmed) {
            this.tokenCreated = false;
            this.showQR = true;
            this.$store.dispatch('relinkTablet', row.id)
              .then(function (res) {
                self.QRText = res.data.token;
                self.tokenCreated = true;
              })
              .catch(function (err) {
                alert('something went wrong')
                console.log(err)
              })
          }
        })

      },
      editTablet(id) {
        this.showEditDeviceId = id;
        this.showEditDeviceModal = true;
      },
      deleteTablet(id) {
        let self = this;
        swal.fire({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          icon: 'warning',
          showCancelButton: true,
          reverseButtons: true,
          confirmButtonColor: '#d62828',
          cancelButtonColor: '#929292',
          confirmButtonText: 'Yes, delete it!'
        }).then((result) => {
          if (result.isConfirmed) {
            self.$store.dispatch('deleteTablet', id)
              .then(function (res) {
                alert('tablet removed from list!');
              })
              .catch(function (err) {
                alert('something went wrong')
                console.log(err)
              })
          }
        })
      },
      showPrinterStatus(id) {
        this.showPrinterStatusDeviceId = id;
        this.showPrinterStatusModal = true;
      },
      closeEditDeviceModal() {
        this.showEditDeviceId = null;
        this.showEditDeviceModal = false;
        document.body.classList.remove("modal-open");
      },
      closePrinterStatusModal() {
        this.showPrinterStatusDeviceId = null;
        this.showPrinterStatusModal = false;
        document.body.classList.remove("modal-open");
      },
    }
  }
</script>

<style scoped>
  a.dropdown-item {
    cursor: pointer;
  }

  .el-dropdown-menu__item {
    line-height: unset;
  }

  .el-dropdown-menu__item:focus, .el-dropdown-menu__item:not(.is-disabled):hover {
    color: #16181b;
    background-color: #f6f9fc;
  }
</style>
