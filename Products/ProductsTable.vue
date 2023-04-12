<template>
  <div>
    <div v-if="showFilter" class="mx-4 mb-4">
      <base-input label="Javascript filtering function (‼️ only visible for admins)" v-model="filterFunction" class="mr-2">
        <InfoIcon interactive slot="appendLabel" icon="fas fa-eye" placement="right" maxWidth="800px"><pre style="white-space:pre-wrap">
          <h5>Write your own filtering function! Examples how to:</h5>
          <ul class="codeExamples">
            <li><code>item => item.price == 21</code><span>//Show all products where item price is equal to 21</span></li>
            <li><code>item => item.categories.length == 0</code><span>//Show all products that don't have a category above them</span></li>
            <li><code>item => item.categories.length > 0 && item.optionGroupParent > 0</code><span>//Show all products and options that are both added to a category AND to an option group as an option.</span></li>
            <li><code>item => item.is_option && item.untill_item_type != 1</code><span>//Show all options that don't have untill_item_type set to 1 (must have option)</span></li>
            <li><code>(item, index, array) => array.filter(item2 => item2.name.toLowerCase() == item.name.toLowerCase()).length > 1</code><span>//Show all items where the kitchen name is the same as another item (potential duplicates)</span></li>
          </ul>
          <h5>The properties below can all be used to filter items in this table</h5>
          "id":24437,
          "restaurant_id":35,
          "id_code":null,
          "name":"Test 21%",
          "price":21,
          "vat":0.21,
          "tags":[],
          "ingredients":null,
          "has_allergens":true,
          "allergens":["crustacean","fish","shellfish"],
          "abv":null,
          "image":"testwerktdit.jpg",
          "card":null,
          "active":true,
          "is_option":false,
          "deal_duration":null,
          "deal_discount":null,
          "deal_enabled_at":null,
          "pos_id":null,
          "untappd_beer_id":null,
          "untappd_beer_rating":null,
          "untappd_beer_description":null,
          "untappd_brewery_name":null,
          "untappd_brewery_logo":null,
          "untappd_brewery_link":null,
          "untappd_brewery_website":null,
          "untappd_brewery_twitter":null,
          "untappd_brewery_facebook":null,
          "untappd_brewery_instagram":null,
          "deal_id":[],
          "keywords":null,
          "untill_item_type":0,
          "translations":[
            {"name":null,
            "description":null,
            "keywords":null,
            "ingredients":null,
            "language":"en"},
            {"name":null,
            "description":null,
            "keywords":null,
            "ingredients":null,
            "language":"nl"}
          ],
          "reservation":false,
          "available_till":null,
          "item_type":
            {"id":6,
            "name":"Nagerechten"},
          "option_groups":[],
          "recommendations":[],
          "disabled_at":null,
          "modified_at":"Wed,29 Dec 2021 23:48:47 GMT",
          "created_at":"2021-01-14T16:25:23.502615",
          "categories":[
            {"id":3680,
            "restaurant_id":35,
            "name":"Hot Drinks copy",
            "active":true,
            "style":"double",
            "quick_select":"hot-drinks",
            ...
            "items":[24948,24947,24949,26189,26190,24950,24951,26186,26187,24437],
            "cards":[]
            },
          "optionGroupParent":[],
          "parent":"parentTest 21%"
        </pre></InfoIcon>
      </base-input>
      <base-button @click="applyFilter">Apply</base-button>
      <base-button @click="resetFilter">Reset</base-button>
    </div>
    <el-table
      v-if="!inactiveProductsLoading || !menuIsLoading"
      ref="multipleTable"
      class="table-responsive table-flush cursor-context-menu sticky-header"
      header-row-class-name="thead-light"
      @row-click="rowClicked"
      :row-class-name="tableRowClassName"
      :data="type === 'inactive' ? inactiveProducts : products"
      @select="handleSelection"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        :label="type === 'options' ? 'Option' : 'Product'"
        min-width="240px"
        prop="name"
        sortable>
        <template v-slot="{row}">
          <div class="media align-items-center">
            <a href="#" class="avatar rounded-circle mr-3">
              <img v-if="row.image" :alt="row.image"
                   :src="'https://orderli.com/cdn-cgi/image/width=100,quality=60/https://imagedelivery.net/xS_5nksgKmcoB2_mcBGUmA/' + row.image"
                   @error="placeholderImgUrl">
            </a>
            <!--            <div class="media-body" v-if="row.translations[0]">-->
            <!--              <h5 class="font-weight-600 name mb-0 text-sm" >{{row.name}}</h5>-->
            <!--              <span v-if="row.translations[0].description" >{{row.translations[0].description}}</span>-->
            <!--            </div>-->
            <div>
              <h5 class="font-weight-600 name mb-0 text-sm">
                {{row.name}}
                <span v-for="translation in row.translations" :key="translation.id">
                  <h6 v-if="languages.find(lang => lang.iso639 === translation.language)" class="mb-0">
                    <span>{{getLanguageEmoji(translation)}}</span>
                    {{translation.name}}
                  </h6>
                </span>
              </h5>

            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="Price"
                       prop="price"
                       width="80px"
                       sortable>
      </el-table-column>

      <el-table-column label="Active"
                       prop="active"
                       width="88px"
                       class-name="no-disabled"
                       sortable>
        <template v-slot="{row}">
                  <span @click.stop>
                    <base-switch class="mr-1" on-text="On" off-text="Off" v-model="row.active"
                                 v-on:input="toggleActive($event, row.id)"></base-switch>
                  </span>
        </template>
      </el-table-column>

      <el-table-column
        v-if="type !== 'options'"
        label="Belongs to category"
        min-width="140px"
        prop="categories"
        sortable>
        <template v-slot="{row}">
          <span v-for="category in row.categories" :key="category.id" class="status badge badge-dot mr-2 my-1"
                style="background: #f6f9fc; padding: 6px; font-size: 0.75rem; border-radius: 10px;box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);">
            {{category.name}}
          </span>
        </template>
      </el-table-column>

      <el-table-column
        v-if="type !== 'products'"
        label="Belongs to option group"
        min-width="140px"
        prop="optionGroupParent"
        sortable>
        <template v-slot="{row}">
          <span v-for="optionGroupId in row.optionGroupParent" :key="row.id + optionGroupId"
                class="status badge badge-dot mr-2 my-1"
                style="background: #f6f9fc; padding: 6px; font-size: 0.75rem; border-radius: 10px;box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);">{{getOptionGroupNameFromMap(optionGroupId)}}</span>
        </template>
      </el-table-column>

      <el-table-column
        :label="type === 'options' ? 'Sub-option groups' :'Option groups'"
        min-width="120px"
        prop="option_groups"
        sortable>
        <template v-slot="{row}">
          <span v-for="option in row.option_groups" :key="option" class="status badge badge-dot mr-2 my-1"
                style="background: #f6f9fc; padding: 6px; font-size: 0.75rem; border-radius: 10px;box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);">{{getOptionFromMap(option).name}}</span>
        </template>
      </el-table-column>

      <el-table-column
        v-if="currentRestaurant.pos_integration"
        label="POS ID"
        prop="pos_id"
        width="115px"
        sortable>
      </el-table-column>

      <el-table-column
        v-if="currentRestaurant.pos_integration === 'untill' && type === 'options'"
        label="ItemType"
        prop="untill_item_type"
        width="110px"
        sortable>
      </el-table-column>

      <el-table-column
        label="Allergens"
        prop="allergens"
        width="180px"
        sortable>
        <template v-slot="{row}">
          <div v-if="row.has_allergens">
            <AllergensIcon v-for="allergen in row.allergens" v-bind:key="allergen.id" v-bind:allergen="allergen"/>
          </div>
          <div v-else><i class="fas fa-check" style="color: #43A046"></i> Allergens Free</div>
        </template>
      </el-table-column>

      <el-table-column
        label="VAT"
        prop="vat"
        width="75px"
        sortable>
        <template v-slot="{row}">
          <span v-if="row.vat">{{row.vat * 100 + '%'}}</span>
        </template>
      </el-table-column>
      <el-table-column
        v-if="this.type !== 'options' && this.currentAccount && this.currentAccount.subscription && this.currentAccount.subscription.features.includes('cross_selling')"
        label="Cross-selling"
        min-width="140px"
        prop="recommendations"
        sortable>
        <template v-slot="{row}">
          <span v-for="item in row.recommendations" :key="item" class="status badge badge-dot mr-2 my-1"
                style="background: #f6f9fc; padding: 6px; font-size: 0.75rem; border-radius: 10px;box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);">
            {{itemsAndOptionsMap.get(item) ? itemsAndOptionsMap.get(item).name : 'recommendation deleted'}}
          </span>
        </template>
      </el-table-column>

      <el-table-column
        label="Type"
        prop="item_type"
        width="115px"
        sortable>
        <template v-slot="{row}">
          <ItemTypeSelect :item="row"/>
        </template>
      </el-table-column>

      <el-table-column label="Last modified"
                       prop="modified_at"
                       width="150px"
                       sortable
                       :sort-method="(a,b) => sortData(a, b, 'modified_at')"
      >
        <template v-slot="{row}">
          <span v-if="row.modified_at">{{formatDate(row.modified_at)}}</span>
          <span v-else>never</span>
          <!--                  {{formatDate(row.modified_at ? row.modified_at : row.created_at)}} TODO make sure modified_at is always present and is set to created_at if never modified-->
        </template>
      </el-table-column>

      <el-table-column min-width="70px"
                       class-name="no-disabled">
        <template v-slot="{row}">
          <el-dropdown trigger="click" class="dropdown">
                    <span @click.stop class="btn btn-sm btn-icon-only text-light">
                      <i class="fas fa-ellipsis-v mt-2"></i>
                    </span>
            <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
              <el-dropdown-item class="dropdown-item" @click.native="editItem(row.id)">Edit {{type === 'options' ?
                'option' : 'item' }}
              </el-dropdown-item>
              <el-dropdown-item class="dropdown-item" @click.native="deleteItem(row.id)">Delete {{type === 'options' ?
                'option' : 'item' }}
              </el-dropdown-item>
              <el-dropdown-item v-if="isAdmin"  class="dropdown-item" @click.native="viewAnalytics(row)">View analytics
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>

          <div @click.stop>
            <EditItemModal v-if="row.id === showItemModal" mode="edit" :isOption="type === 'options'" :itemData="row"
                           :show="row.id === showItemModal && showModalDelayed" v-on:modalClose="closeEditItemModal"/>
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
</template>

<script>
  import swal from 'sweetalert2';
  import {mapGetters} from 'vuex';
  import 'sweetalert2/dist/sweetalert2.css';
  import EditItemModal from "@/views/Products/EditItemModal";
  import AllergensIcon from '@/views/Products/AllergensIcon.vue';
  import ItemTypeSelect from "@/components/Inputs/ItemTypeSelect";
  import {Table, TableColumn, DropdownMenu, DropdownItem, Dropdown} from 'element-ui';

  export default {
    name: 'ProductsTable',
    components: {
      [Table.name]: Table,
      [TableColumn.name]: TableColumn,
      [Dropdown.name]: Dropdown,
      [DropdownItem.name]: DropdownItem,
      [DropdownMenu.name]: DropdownMenu,
      EditItemModal,
      AllergensIcon,
      ItemTypeSelect,

    },
    props: ["type", "showFilter", "inactiveProducts", "inactiveProductsLoading"],
    data() {
      return {
        doesTakeawayOrDelivery: true,
        showNewItemModal: false,
        showModalDelayed: false,
        multipleSelection: [],
        lastSelectionItem: null,
        lastSelectionWasGrowing: false,
        filterFunction: "item => true",
        appliedFilterFunction: (item => true),
        shiftIsPressed: false,
        lastSelectionChangeManual: true,
      };
    },
    created() {
      this.$store.dispatch('getMenuIfUndefined');
      document.onkeydown = this.onKeyDown;
      document.onkeyup = this.onKeyUp;
    },
    computed: {
      isAdmin() {
        return this.$store.getters.isAdmin;
      },
      ...mapGetters({
        // products: ,
        // categoriesMap: 'categoriesMap',
        // cardsMap: 'cardsMap',
        optionGroupsMap: 'optionGroupsMap',
        menuIsLoading: 'menuIsLoading',
        inactiveItemsIsLoading: 'inactiveItemsIsLoading',
        currentRestaurant: 'currentRestaurant',
        currentAccount: 'currentAccount',
        itemsAndOptionsMap: 'itemsAndOptionsMap'
      }),
      products() {
        return this.$store.getters[`${this.type === 'options' ? 'optionItemsOfCurrentRestaurant' : (this.type === 'all' ? 'itemsAndOptionItemsOfCurrentRestaurant' : 'itemsOfCurrentRestaurant')}`].filter(this.appliedFilterFunction);
      },
      languages() {
        return this.$store.state.languages.filter(language => this.currentRestaurant?.languages.find(lang => language.iso639 === lang))
      },
      showItemModal: {
        get() {
          return this.$store.state.menu.showItemModalId;
        },

        set(show) {
          this.$store.commit('setItemModal', show);
          if (!show) {
            this.showModalDelayed = false
          }
        }
      },
    },
    methods: {
      viewAnalytics(row) {
        let idArray = this.products.map(element => element.id)
        let nameArray = this.products.map(element => element.name)

        localStorage.setItem('menuItemsOrder', idArray);
        localStorage.setItem('menuItemsOrderNames', JSON.stringify(nameArray));

        this.$router.push(`/analytics/item/${row.id}`)
      },
      onKeyDown() {
        let key = window.event.keyCode;
        if (key === 16) {
          this.shiftIsPressed = true;
        }
      },
      onKeyUp() {
        this.shiftIsPressed = false;
      },
      newItemModal(value) {
        this.showNewItemModal = value;
      },
      rowClicked(row) {
        this.editItem(row.id);
      },
      handleSelection(selection, row){
        //Determine if the click was checking or unchecking a checkbox
        let selectionIsGrowing = this.multipleSelection.length < selection.length

        if(this.lastSelectionWasGrowing === selectionIsGrowing && this.shiftIsPressed){
          //A checkbox was checked/unchecked while holding shift, previous action was the same so either delete or add all checkboxes in between the items.

          //get the last item of the previous array
          let prevItem = this.lastSelectionItem;
          let currenItem = row;

          //get other info that's needed
          let allItems = this.type === 'inactive' ? this.inactiveProducts : this.products
          let allItemsFlat = allItems.map(item => item.id);
          let prevItemIndex = allItemsFlat.indexOf(prevItem.id);
          let currentItemIndex = allItemsFlat.indexOf(currenItem.id);
          let highestIndex = prevItemIndex < currentItemIndex ? currentItemIndex : prevItemIndex;
          let lowestIndex = prevItemIndex > currentItemIndex ? currentItemIndex : prevItemIndex;
          let itemsToSelect = allItems.slice(lowestIndex, highestIndex + 1);
          this.lastSelectionChangeManual = false;

          //already update the values - necessary because shift+click adding 1 item doesn't trigger handleSelectionChange
          this.multipleSelection = selection;
          this.$emit('multipleSelect', selection);

          //loop through all the items to bulk select and set them to true/false depending on boolean value, this trigger handleSelectionChange if there is an actual change
          itemsToSelect.forEach(item => this.$refs.multipleTable.toggleRowSelection(item, selectionIsGrowing))
          this.lastSelectionChangeManual = true;
        }else{
          //Only update multipleSelection if it was triggered by a non-shift click, else handleSelectionChange will trigger the change
          this.lastSelectionWasGrowing = selectionIsGrowing;
          this.lastSelectionItem = row;
          this.multipleSelection = selection;
          this.$emit('multipleSelect', selection);
        }


      },
      handleSelectionChange(selection) {
        if(!this.lastSelectionChangeManual) {
          //the shift click also triggers this function, prevent a continuous loop, but do log the new length of the selection by updating multipleSelection
          this.multipleSelection = selection;
          this.$emit('multipleSelect', selection);
        }

      },
      editItem(id) {
        this.showItemModal = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
      },
      uploadImage(id) {
        alert('want to change image of item ' + id + '. Uploading or changing images is not yet supported and still a work in progress')
      },
      closeEditItemModal() {
        this.$store.commit('setItemModal', null);
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      getOptionGroupNameFromMap(id) {
        return this.optionGroupsMap.get(id)?.name;
      },
      getOptionFromMap(id) {
        let option = this.optionGroupsMap.get(id);
        if (option && option.name) {
          return this.optionGroupsMap.get(id);
        } else {
          return {name: 'OPTION GROUP NOT FOUND'}
        }
      },
      deleteItem(id) {
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
            self.$store.dispatch('deleteItem', id)
              .then(function () {
                //TODO trigger notification of succesful deletion
              })
              .catch(function (err) {
                alert(err)
              })
          }
        })
      },
      toggleActive(event, id) {
        //Dispatch PUT event only for changed active settings
        //TODO group this events and only send the last data when user clicks fast multiple times
        let itemsArr = [];
        itemsArr.push({'id': id, 'active': event});
        if (this.type === 'inactive') {
          this.$store.dispatch('updateInactiveItem', itemsArr)
            .then(function () {
              console.log('item updated');
            })
            .catch(function (err) {
              alert(err)
            })
        } else {
          this.$store.dispatch('updateItem', itemsArr)
            .then(function () {
              console.log('item updated');
            })
            .catch(function (err) {
              alert(err)
            })
        }
      },
      tableRowClassName({row, rowIndex}) {
        if (!row.active) {
          return 'inactive-item';
        }
        return '';
      },
      placeholderImgUrl(event) {
        let placeholderSrc = "/img/menu/image-placeholder-350x350.png";//http://www.stleos.uq.edu.au/wp-content/uploads/2016/08/image-placeholder-350x350.png";
        if (event.srcElement.currentSrc == placeholderSrc) {
          console.log("we got into a loop");
        } else {
          event.target.src = placeholderSrc;
        }

      },
      formatDate(dateString) {
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
      applyFilter(){
        this.appliedFilterFunction = eval(this.filterFunction);
      },
      resetFilter(){
        this.filterFunction = "item => true";
        this.appliedFilterFunction = (item => true);
      },
      sortData(a,b,key){
        console.log('busy sorting');
        const nullDate = new Date(0)

        let dateA = a[key] ? new Date(a[key]) : nullDate
        let dateB = b[key] ? new Date(b[key]) : nullDate
        return dateA.getTime() - dateB.getTime()
      },
      getLanguageEmoji(translation) {
        return this.languages.find(lang => translation.language === lang.iso639)?.emoji
      },
    },
  }
</script>
<style scoped>
  .cell {
    overflow: visible;
  }

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

  .codeExamples{
    display: flex;
    flex-direction: column;
  }

  .codeExamples code{
    background-color: #e5e5e5;
    color: #d64f28;
    padding: 4px 6px;
    overflow:hidden;
    border-radius: 4px;
    margin-right: .5rem;
  }

  .codeExamples span{
    color: #808080;
    user-select: none;
    line-height: 28px;
  }
</style>
