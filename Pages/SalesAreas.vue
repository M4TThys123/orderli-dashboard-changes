<template>
  <div class="content">
    <div class="container-fluid mt-4">
      <div>
        <div @click.stop>
          <GenerateQRCodesModal :show="showGenerateQRCodes" v-on:modalClose="showGenerateQRCodes = false"/>
          <EditQRModal v-if="allQRs" mode="new" :show="showNewQRModal" v-on:modalClose="showNewQRModal = false"/>
          <EditQRModal v-if="allQRs" mode="bulk" :show="showBulkQRModal" v-on:modalClose="showBulkQRModal = false" :bulkItems="multipleSelection"/>
        </div>
        <div class="card d-none d-md-block" footer-classes="pb-2">
          <div class="border-0 card-header">
            <h3 class="mb-0 pr-4 d-inline-block">Manage all QR codes</h3>
            <base-button v-if="isAdmin" @click="showNewQRModal = true" class="px-3" type="primary"><i class="fa fa-plus pr-2" style="font-size: .8rem;"></i>New QR Code (admin)</base-button>
            <base-button @click="showGenerateQRCodes = true" class="px-3" type="primary"><i class="fa fa-qrcode pr-2" style="font-size: .8rem;"></i>Generate QR codes</base-button>

            <base-input class="ml-auto mr-1"  label="Show QR Codes">
              <base-switch label="Show QR Codes" on-text="On" off-text="Off" v-model="showQRcodes"></base-switch>
            </base-input>

            <base-dropdown v-if="isAdmin" style="margin-left: 1rem;" title-classes="btn btn-primary">
              <template v-slot:title>
                Bulk actions (admin)
              </template>
              <a class="dropdown-item" href="#" @click="bulkEdit" >Edit {{multipleSelection.length}} selected QRs</a>
              <a class="dropdown-item" href="#" disabled @click="bulkMove" >Move {{multipleSelection.length}} QRs to another restaurant</a>
              <a class="dropdown-item" href="#" disabled @click="bulkDelete" >Delete {{multipleSelection.length}} selected QRs</a>
            </base-dropdown>
          </div>
          <el-table
              v-if="allQRs"
              :data="allQRs"
              @row-click="rowClicked"
              @selection-change="handleSelectionChange"
            >
              <el-table-column
                type="selection"
                width="55">
              </el-table-column>
            <el-table-column
              v-if="showQRcodes"
              width="200">
              <template v-slot="{row}">
                <VueQRCodeComponent class="QRCodeComponent" :text="`${serverUrl}${row.code}`" :size="150" color="#172b4d" bg-color="#fff" errorLevel="L"></VueQRCodeComponent>
              </template>
            </el-table-column>

              <el-table-column
                prop="code"
                label="QR Code"
                sortable
                min-width="120">
                <template v-slot="{row}">
                  {{row.code}}
                  <a :href="`${serverUrl}${row.code}`" target="_blank" class="">
                    <InfoIcon icon="fas fa-external-link-alt">Open in new window</InfoIcon>
                  </a>

                </template>
              </el-table-column>
              <el-table-column
                prop="table_nr"
                label="Table NR"
                sortable
                min-width="100">
              </el-table-column>
              <!--            <el-table-column-->
              <!--              prop="manual_table_nr"-->
              <!--              label="Manual Table Nr"-->
              <!--              sortable-->
              <!--              width="140px"-->
              <!--            >-->
              <!--              <template v-slot="{row}">-->
              <!--                <div v-if="row.manual_table_nr"><i class="far fa-check-circle"></i> Yes</div>-->
              <!--              </template>-->
              <!--            </el-table-column>-->
              <el-table-column label="Manual Table Nr"
                               prop="manual_table_nr"
                               width="88px"
                               class-name="no-disabled"
                               sortable>
                <template v-slot="{row}">
                  <span @click.stop>
                    <base-switch class="mr-1" on-text="On" off-text="Off" v-model="row.manual_table_nr" v-on:input="toggleActive($event, row)"></base-switch>
                  </span>
                </template>
              </el-table-column>
              <el-table-column
                prop="active"
                label="Active"
                sortable
                width="100">
                <template v-slot="{row}">
                <span @click.stop>
                    <base-switch class="mr-1" on-text="On" off-text="Off" v-model="row.active" v-on:input="toggleActive($event, row)"></base-switch>
                  </span>
                </template>
              </el-table-column>
              <el-table-column
                prop="color"
                label="Area"
                sortable
                width="130"
              >
              </el-table-column>
              <el-table-column
                prop="takeaway"
                label="Takeaway"
                sortable
                width="140px"
              >
                <template v-slot="{row}">
                  <div v-if="row.takeaway"><i class="far fa-check-circle"></i> Yes</div>
                </template>
              </el-table-column>
              <el-table-column
                prop="takeaway_onsite"
                label="To Go Ordering"
                sortable
                width="160px"
              >
                <template v-slot="{row}">
                  <div v-if="row.takeaway_onsite"><i class="far fa-check-circle"></i> Yes</div>
                </template>
              </el-table-column>
              <el-table-column
                prop="delivery"
                label="Delivery"
                sortable
                width="140px"
              >
                <template v-slot="{row}">
                  <div v-if="row.delivery"><i class="far fa-check-circle"></i> Yes</div>
                </template>
              </el-table-column>

              <el-table-column width="90px"
                               class-name="no-disabled">
                <template v-slot="{row}">
                  <el-dropdown trigger="click" class="dropdown">
                    <span @click.stop class="btn btn-sm btn-icon-only text-light">
                      <i class="fas fa-ellipsis-v mt-2"></i>
                    </span>
                    <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                      <el-dropdown-item class="dropdown-item" @click.native="editQR(row)">Edit QR</el-dropdown-item>
                      <el-dropdown-item class="dropdown-item" @click.native="deleteQR(row)">Remove QR</el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>

                  <div @click.stop>
                    <EditQRModal v-if="row.id === showQRModalId" mode="edit" :qrData="row" :show="row.id === showQRModalId && showModalDelayed" v-on:modalClose="closeEditQRModal"/>
                  </div>

                </template>
              </el-table-column>

            </el-table>
          <div v-else class="d-flex justify-content-center py-8">
            <div class="spinner-border text-primary" role="status">
              <span class="sr-only">Loading...</span>
            </div>
          </div>
        </div>
        <div class="d-md-none">
          <div class="controls d-flex align-items-center mb-5">
            <el-select v-model="sortBy" placeholder="Sort by ...">
              <el-option
                v-for="item in sortOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
            <div @click="sortAscending = !sortAscending" style="margin-left: 1rem; width: auto; height: 100%; padding: 8px 12px; border-radius: 4px; background-color: white; border: solid 1px #DCDFE6">
              <i class="fas fa-arrow-down" :class="sortAscending ? 'fa-arrow-down' : 'fa-arrow-up'"></i>
            </div>
          </div>
          <div
              v-if="allQRs"
              :key="table.id"
              v-for="table in allQRsMobile"
            >
              <div
                @click.stop="editQR(table)"
                style="min-height: 40px; padding: 14px;"
                :style="{'border-left': `24px solid ${areas.find(area => area.value === table.color).color || 'white'}`}"
                class="card mobile-card flex-row justify-content-between align-items-center"
                :class="{'disabled': !table.active}"
              >
                <!--              <pre>{{table}}</pre>-->
                <h2 style="margin: 0">{{table.table_nr}}</h2>
                <!--                {{table.code}}-->
                <a :href="`${serverUrl}${table.code}`" target="_blank" class="ml-2 mr-auto">
                  <InfoIcon icon="fas fa-external-link-alt">Open in new window</InfoIcon>
                </a>
                <base-switch @click.native.stop="" class="toggle mr-1" on-text="On" off-text="Off" v-model="table.active" v-on:input="toggleActive($event, table)"></base-switch>
                <!--              <div class="d-flex justify-content-between align-items-center" style="height: 100%">-->
                <!--                -->
                <!--              </div>-->


              </div>
              <div @click.stop>
                <EditQRModal v-if="table.id === showQRModalId" mode="edit" :qrData="table" :show="table.id === showQRModalId && showModalDelayed" v-on:modalClose="closeEditQRModal"/>
              </div>
            </div>
          <div v-else class="d-flex justify-content-center py-8">
            <div class="spinner-border text-primary" role="status">
              <span class="sr-only">Loading...</span>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>



</template>

<script>
import {Dropdown, DropdownItem, DropdownMenu, Table, TableColumn} from "element-ui";
import swal from "sweetalert2";
import EditQRModal from "@/views/Pages/EditQRModal";
import GenerateQRCodesModal from "@/views/Pages/GenerateQRCodesModal";
import VueQRCodeComponent from 'vue-qrcode-component'

export default {
  name: "SalesAreas",
  data() {
    return {
      showModalDelayed: false,
      showQRModalId: null,
      showNewQRModal: false,
      showBulkQRModal: false,
      showGenerateQRCodes: false,
      multipleSelection: [],
      sortBy: 'table_nr_sortable',
      sortOptions: [
      {
        value: 'table_nr_sortable',
        label: 'Sort by table number'
      }, {
        value: 'color',
        label: 'Sort by area'
      }, {
        value: 'active',
        label: 'Sort by disabled'
      }],
      sortAscending: true,
      showQRcodes: false,
    }
  },
  components: {
    [Table.name]: Table,
    [TableColumn.name]: TableColumn,
    [Dropdown.name]: Dropdown,
    [DropdownItem.name]: DropdownItem,
    [DropdownMenu.name]: DropdownMenu,
    EditQRModal,
    GenerateQRCodesModal,
    VueQRCodeComponent
  },
  mounted(){
    if(this.allQRs.length < 1){
      this.$store.dispatch('getQRs')
    }
  },
  computed: {
    allQRs() {return this.$store.getters.allQRsOfCurrentRestaurant},
    allQRsMobile() {
      let arrCopy = [...this.allQRs];
      if(this.sortBy === 'table_nr_sortable'){
        let longestTableNr = arrCopy.reduce((a, b) => a.table_nr?.length > b.table_nr?.length ? a : b, 0).table_nr?.length;
        const reg = /([0-9.]+)(?![0-9.])|([a-z]+)(?![a-z])/gi;

        arrCopy.forEach(item => {
          let tableNrLowercase = null;
          item.table_nr_sortable = null;
          if(item.table_nr) {
            tableNrLowercase = item.table_nr.toLowerCase();
            let tableNrSplit = tableNrLowercase.match(reg);
            tableNrSplit = tableNrSplit.map(split => isNaN(split) ? split : split.padStart(longestTableNr, '0'));
            item.table_nr_sortable = tableNrSplit.join('')
          }

        })
      }

      return arrCopy.sort((a, b) =>
      {
        if (a[this.sortBy] === null) return this.sortAscending ? 1 : -1;
        if (b[this.sortBy] === null) return this.sortAscending ? -1 : 1;
        if (a[this.sortBy] === b[this.sortBy]) return 0;

        return ( a[this.sortBy] < b[this.sortBy] ) ? (this.sortAscending ? -1 : 1) : (this.sortAscending ? 1 : -1);

      })
    },
    isAdmin() {return this.$store.getters.isAdmin},
    areas() {return this.$store.state.areaColors},
    serverUrl() {return process.env.VUE_APP_SERVER_URL || 'https://web.orderli.com/'},
  },
  methods: {
    editQR(row){
      this.showQRModalId = row.id;
      this.$nextTick(function () {
        this.showModalDelayed = true;
      })
    },
    deleteQR(row){
      let self = this;
      swal.fire({
        title: 'Are you sure?',
        text: "You won't be able to revert this!",
        icon: 'warning',
        showCancelButton: true,
        reverseButtons: true,
        confirmButtonColor: '#d62828',
        cancelButtonColor: '#929292',
        confirmButtonText: 'Remove QR'
      }).then((result) => {
        if (result.isConfirmed) {
          self.$store.dispatch('deleteQR', row)
            .catch(function(err) {
              alert(err)
            })
        }
      })
    },
    rowClicked(row){
      this.editQR(row);
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    closeEditQRModal(){
      this.showQRModalId = null;
      this.showModalDelayed = false;
      document.body.classList.remove("modal-open");
    },
    toggleActive(event, row){
      //Dispatch PUT event only for changed active settings
      //TODO group this events and only send the last data when user clicks fast multiple times
      let qrObj = row;
      this.$store.dispatch('updateQR', qrObj)
        .then(function(){
          console.log('QR updated');
        })
        .catch(function(err) {
          alert(err)
        })
    },
    bulkEdit(){
      this.showBulkQRModal = true;
    },
    bulkMove(){
      alert('action not supported yet, no endpoint');

    },
    bulkDelete(){
      this.multipleSelection.forEach((QR, index) => {
        setTimeout(() => {
          this.$store.dispatch('deleteQR', QR)
            .catch(function(err) {
              alert(err)
            })
        },  index * 100);
      })
    }

  }
}
</script>

<style scoped>

.mobile-card.disabled .toggle /deep/ .custom-toggle-slider{
  border-color: #424242;

}

.mobile-card.disabled .toggle /deep/ .custom-toggle-slider:before{
  background-color: #424242;
}

.mobile-card.disabled .toggle /deep/  .custom-toggle-slider:after{
  color: #424242;
}

.QRCodeComponent /deep/ img{
  mix-blend-mode: darken;
}

</style>
