<template>
  <modal class="editItemModal" v-if="item" :size="canManageAccounts ? 'xl' : 'lg'" :show="show" v-on:update:show="onClose" bodyClasses="py-0" headerClasses="border-bottom" footer-classes="border-top">
    <template slot="header">
      <template v-if="isOption">
        <h5 v-if="mode === 'edit'" class="modal-title">Editing option</h5>
        <h5 v-else class="modal-title">{{addToParentId ? 'New option in ' + getNameFromOptionGroupsMap(addToParentId) :
          'New option' }}</h5>
      </template>
      <template v-else>
        <h5 v-if="mode === 'edit'" class="modal-title">Editing product</h5>
        <h5 v-else class="modal-title">{{addToParentId ? 'New product in ' + getNameFromCategoryMap(addToParentId) :
          'New product' }}</h5>
      </template>
    </template>
    <div class="row mx--4">
      <!--      <pre>-->
      <!--        Beta, still todo:-->
      <!--        - complete image upload-->
      <!--        - add category multi-select with search embedded-->
      <!--        - Add order_ahead toggle-->
      <!--        - Add available till-->
      <!--        - Add deal_id-->
      <!--        - Add keywords-->
      <!--        - add ingredients-->
      <!--      </pre>-->

      <form :class="[canManageAccounts && item.pos_id && currentRestaurant.pos_integration ? 'col-9' : 'col-12']" class="py-4" @keyup.enter.prevent="onsubmit" @submit.prevent="onsubmit">
        <div>
          <div class="row">
            <div class="col-lg-6">
              <base-input
                type="text"
                required
                ref="autofocus"
                name="name"
                :label="currentRestaurant.languages.length > 1 ? 'Kitchen Name' : 'Name'"
                :placeholder="mode === 'new' ? 'Coca cola 0,2L' : null"
                v-model="item.name"
              >
              </base-input>
            </div>
            <div class="col-lg-6">
              <base-input ref="priceField" label="Price">
                <currency-input
                  ref="input"
                  v-model.lazy="item.price"
                  class="form-control"
                  :currency="currentRestaurant.currency"
                  locale="nl"
                />
              </base-input>
            </div>
          </div>
          <div v-if="item.price_levels && item.price_levels.length" class="row">
            <div v-for="priceLevel in item.price_levels" class="col-lg-6">
              <base-input :label="priceLevel.price_id ? 'Price level for ' + priceLevel.price_id : priceLevel.takeaway_price ? 'Takeaway price' : priceLevel.delivery_price ? 'Delivery price' : ''">
                <currency-input
                  ref="input"
                  v-model.lazy="priceLevel.amount"
                  class="form-control"
                  :currency="currentRestaurant.currency"
                  locale="nl"
                />
              </base-input>
            </div>
          </div>
          <!--          <div v-if="currentRestaurant.languages.length <= 1" class="row">-->
          <!--            <div class="col-12">-->
          <!--              <base-input label="Description">-->
          <!--                <textarea rows="4" class="form-control" v-model.lazy="item.translations.find(x => x.language === currentRestaurant.languages[0]).description" :placeholder="mode === 'new' ? 'Coca-Cola Classic is the world\'s favourite soft drink and has been enjoyed since 1886.' : null"></textarea>-->
          <!--              </base-input>-->
          <!--            </div>-->
          <!--          </div>-->


          <div class="row" :class="{'justify-content-between': currentRestaurant.pos_integration}">
            <div class="col-lg-auto">

              <base-input label="Active">
                <el-tooltip :disabled="item.active || !item.disabled_at" :content="'Last disabled ' + disabledAtDate"
                            placement="right">
                  <base-switch class="mr-1" on-text="On" off-text="Off" v-model="item.active"></base-switch>
                </el-tooltip>
              </base-input>
            </div>
            <div
              class="col-lg-auto"
              v-if="currentRestaurant.pos_integration"
            >
              <label class="form-control-label">{{ currentRestaurant.pos_integration.toLowerCase() === 'vectron' ? 'PLU'
                : 'POS ID' }}
                <el-select
                  v-model="posSearch"
                  filterable
                  remote
                  reserve-keyword
                  value-key="pos_id"
                  placeholder="Search in POS"
                  :remote-method="searchPos"
                  :loading="posSearchLoading">
                  <el-option
                    v-for="(item, index) in posSearchResults"
                    :key="item.pos_id + index"
                    :label="item.name"
                    :value="item">
                    <tippy :delay="[1000,0]" animation="fade" :duration="0" :animationFill="false" boundary="window"
                           placement="top-start" theme="no-padding">
                      <template v-slot:trigger>
                        <span style="float: left">{{ item.is_option ? '(M️)' : '(P) '  + item.pos_id + ' | ' + item.name }}</span><span
                        style="float: right; color: #8492a6; font-size: 13px">{{ item.price }}</span>
                      </template>
                      <div :key="optionGroup.pos_id + item.pos_id" v-for="(optionGroup, index) in item.option_groups">
                        <b>{{optionGroup.name}}</b>
                        <div :key="option.pos_id + optionGroup.pos_id + item.pos_id"
                             v-for="option in optionGroup.options">{{ option.name }}
                        </div>
                        <br>
                      </div>
                    </tippy>
                  </el-option>
                </el-select>
              </label>
              <base-input
                type="text"
                ref="posIdField"
                name="POS ID"
                :placeholder="mode === 'new' ? '1234567' : null"
                v-model.lazy="item.pos_id"
              >
              </base-input>
            </div>

            <div
              class="col-lg-auto"
              v-if="currentRestaurant.pos_integration === 'untill'"
            >
              <base-input
                label="OrderItemType"
              >
                <InfoIcon icon="fas fa-exclamation-triangle" slot="appendLabel">Please set the Untill item type by editing the option group. Untill item type on single products will be deprecated soon.</InfoIcon>
                <el-select
                  v-model.number="item.untill_item_type"
                  class="select-danger"
                >
                  <el-option
                    v-for="subscription in [{value: 0, label: '0 Normal article (either regular or menu)'},{value: 1, label: '1 Must have option'},{value: 2, label: '2 Free option'},{value: 3, label: '3 Supplement'},{value: 4, label: '4 Condiment'},{value: 5, label: '5 Menu item'}]"
                    :key="subscription.value"
                    :label="subscription.label"
                    :value="subscription.value">
                  </el-option>
                </el-select>
              </base-input>
            </div>

            <div
              class="col-lg-auto"
              v-if="currentRestaurant.pos_integration === 'untill' || currentRestaurant.pos_integration === 'lightspeed' || canManageAccounts">
              <base-input label="Option item">
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="item.is_option"></base-switch>
              </base-input>
              <base-input label="Error item" v-if="currentRestaurant.pos_integration === 'untill'">
                <base-switch class="mr-1" on-text="On" off-text="Off" v-model="item.error_item"></base-switch>
              </base-input>
            </div>

            <div class="col-lg-auto">
              <base-input ref="vatField" label="VAT">
                <button-radio-group :options="VATOptions"
                                    class="vat-radio-button-group"
                                    button-classes="btn-secondary vat-radio-button"
                                    v-model.number="item.vat">
                </button-radio-group>
              </base-input>
            </div>

            <div class="col-lg-auto">
              <base-input label="Type">
                <ItemTypeSelect :item="item" changeOnSave v-on:input="item.item_type_id = $event"/>
              </base-input>
            </div>
          </div>

          <div class="row">
            <div class="col-12">
              <label class="form-control-label">Stock</label>
              <InfoIcon>The amount of this item currently in inventory. The stock automatically decreases once an item is ordered. An item becomes out of stock once the stock has decreased to 0. If the stock field is left empty, the item has an unlimited supply.</InfoIcon>
              <base-input type="text" name="stock" v-model.lazy="item.stock" />
            </div>
          </div>

          <div class="row">
            <div class="col-12">
              <label class="form-control-label">
                <i class="fas fa-lock"
                   v-if="currentAccount.subscription === null ||
             !currentAccount.subscription.features.includes('up_selling')"></i>
                {{isOption ? 'Sub-option groups' : 'Option Groups'}}
                <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                       v-if="currentAccount.subscription === null ||
                 !currentAccount.subscription.features.includes('up_selling')">STANDARD</badge>
                <base-button v-if="!isOption" size="sm" class="ml-2"
                             @click.stop="currentAccount.subscription === null ||
                       !currentAccount.subscription.features.includes('up_selling') ?
                       showUpgradeModal('up_selling') : newOptionGroupModal(true)"
                             type="primary">
                  <i class="fa fa-plus pl-0 pr-1" style="font-size: .7rem;"></i>
                  New option group
                </base-button>
                <InfoIcon v-else icon="fas fa-exclamation-triangle">Adding options here will become sub-options! <br>E.g.
                  the option group 'Do you want fries with that?' can have the option 'Yes' and 'No'. The option 'Yes'
                  can then be linked to an option group 'Which sauce do you want on the fries?' using this field.
                </InfoIcon>
              </label>
              <base-input>
                <el-select
                  v-model="item.option_groups"
                  filterable
                  multiple
                  :placeholder="dropdownVisible ? 'Search option groups...' : 'Add option groups'"
                  v-on:visible-change="dropdownVisible = $event"
                  class="select-danger"
                  :disabled="currentAccount.subscription === null ||
            !currentAccount.subscription.features.includes('up_selling')"
                >
                  <el-option
                    v-for="optionGroup in allOptionGroups"
                    :key="optionGroup.id"
                    :label="optionGroup.name"
                    :value="optionGroup.id">
                    <tippy :delay="[1000,0]" animation="fade" :duration="0" :animationFill="false" boundary="window"
                           placement="top-start" theme="no-padding">
                      <template v-slot:trigger><span>{{optionGroup.name}} ({{optionGroup.items.length}})</span>
                      </template>
                      <div :key="translation.language" v-for="translation in optionGroup.translations">
                        [{{translation.language}}] {{translation.title}}<span v-if="translation.description"> - {{translation.description}}</span>
                      </div>
                    </tippy>
                  </el-option>
                </el-select>
              </base-input>
            </div>

            <div class="col-12">
              <collapse>
                <draggable
                  class="list-group"
                  tag="ul"
                  v-model="item.option_groups"
                  v-bind="dragOptionsItem"
                  @start="drag = true"
                  @end="drag = false"
                  handle=".handle"
                >
                  <transition-group type="transition" :name="!drag ? 'flip-list' : null">
                    <collapse-item
                      v-for="(optionGroup, index) in item.option_groups"
                      :key="optionGroup"
                      :id="optionGroup.toString()">
                      <template slot="title">
                        <div class="handle" v-if="item.option_groups.length > 1">
                          <i class="fa fa-grip-vertical"></i>
                        </div>
                        <h5 class="mb-0 ml-2" style="line-height: 28px; background: unset;">{{
                            getNameFromOptionGroupsMap(optionGroup) }}
                          <base-button type="primary" size="sm" class="float-right mr-4"
                                       @click="editOptionGroup(optionGroup)">Edit
                          </base-button>
                        </h5>
                        <EditOptionGroupModal @click.native.stop="" v-if="show && optionGroup === showOptionGroupModalId"
                                              mode="edit" :itemData="item" :optionData="getFromOptionGroupsMap(optionGroup)"
                                              :show="optionGroup === showOptionGroupModalId && showModalDelayed"
                                              v-on:modalClose="closeEditOptionGroupModal"/>
                      </template>
                      <div>
                        <div :key="option" v-for="option in getItemsFromOptionGroupsMap(optionGroup)">
                          <div class="d-flex align-items-center justify-content-between my-2 mr-2 p-2">
                            <small>• {{getNameFromItemsAndOptionsMap(option)}}</small>
                            <base-button size="sm" type="secondary" @click.native="editOptionItem(option)">Edit
                            </base-button>
                          </div>
                          <EditItemModal @click.native.stop="" v-if="show && option === showOptionItemModalId" mode="edit"
                                         :isOption="getIsOptionFromItemsAndOptionsMap(option)"
                                         :itemData="getFromItemsAndOptionsMap(option)"
                                         :show="option === showOptionItemModalId && showModalDelayed"
                                         v-on:modalClose="closeEditOptionItemModal"/>
                        </div>
                      </div>
                    </collapse-item>
                  </transition-group>
                </draggable>

              </collapse>
            </div>


            <div v-if="!isOption" class="col-12">
              <label class="form-control-label">
                <i class="fas fa-lock"
                   v-if="currentAccount.subscription === null ||
             !currentAccount.subscription.features.includes('cross_selling')"></i>
                Cross-selling
                <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                       v-if="currentAccount.subscription === null ||
                 !currentAccount.subscription.features.includes('cross_selling')">PRO</badge>
              </label>
              <base-input>
                <el-select
                  v-model="item.recommendations"
                  filterable
                  multiple
                  placeholder="Search cross-selling-items..."
                  class="select-danger"
                  :disabled="currentAccount.subscription === null ||
            !currentAccount.subscription.features.includes('cross_selling')"
                >
                  <el-option
                    v-for="item in allItems"
                    :key="item.id"
                    :label="`${item.name} (${item.price})`"
                    :title="item.translations"
                    :value="item.id">
                  </el-option>
                </el-select>
              </base-input>
            </div>

            <div class="col-12">
              <label class="form-control-label">Allergens</label>
              <!-- <base-switch class="mr-1" on-text="Yes" off-text="No" v-model="item.has_allergens"></base-switch> -->
              <div>
          <span @click="item.has_allergens = !item.has_allergens" class="badge badge-dot mr-2 my-1"
                v-bind:class="[item.has_allergens ? 'allergens-button allergens-color' : 'allergens-button allergens-active-color']">No Allergens</span>
                <span v-for="allergen in this.allergens" :key="allergen.id"
                      v-bind:class="[(checkAllergens(allergen.value) ? 'allergens-active-color' : 'allergens-color'),
                (item.has_allergens ? '' : 'allergens-inactive')]"
                      @click="changeAllergens(allergen.value)"
                      class="allergens-button badge badge-dot mr-2 my-1">{{allergen.label}}
          </span>
              </div>
            </div>
          </div>

          <EditOptionGroupModal v-if="show && !isOption" mode="new" :show="showNewOptionGroupModal"
                                v-on:modalClose="newOptionGroupModal(false)"
                                v-on:createdOptionGroupId="item.option_groups.push($event)"/>

          <template v-if="currentRestaurant.languages.length > 0">
            <h6 class="heading-small text-muted mt-3">Translations</h6>
            <div ref="translations" class="row">
              <div
                :class="[item.translations.length > 1 ? 'col-lg-6' : 'col-lg-12']"
                v-for="(translatedItem, index) in item.translations"
              >
                <base-input
                  :ref="'translationName' + index"
                  class="translationName"
                  type="text"
                  :label="translatedItem.language.toUpperCase() + ' name'"
                  :placeholder="mode === 'new' ? 'Coca cola 0,2L' : null"
                  v-model.lazy="translatedItem.name"
                >
                </base-input>
                <base-input
                  :ref="'description' + index"
                  :label="translatedItem.language.toUpperCase() + ' description'">
            <textarea
              @keyup.enter.prevent="$event.stopPropagation()"
              rows="4"
              class="form-control description"
              v-model.lazy="translatedItem.description"
              :placeholder="mode === 'new' ? 'Coca-Cola Classic is the world\'s favourite soft drink and has been enjoyed since 1886.' : null"
            ></textarea>
                </base-input>
              </div>
            </div>
          </template>

          <div v-if="!isOption">
            <label class="form-control-label">Temporary discount</label>
            <InfoIcon>
              A limited time discount for this item, the discount is activated on the tablet via the menu editor
              (Tablet menu editor needs to be enabled).
            </InfoIcon>
            <base-input
              type="number"
              v-model="item.deal_discount">
              <small slot="append" class="font-weight-bold">
                %
              </small>
            </base-input>
          </div>

          <div v-if="!isOption">
            <label class="form-control-label">Deal items</label>
            <InfoIcon>Link other items to this item as a deal. If the linked items (deals) are active on the menu, this
              item won't be shown on the menu but instead will be overwritten by the deal item and get a golden border.
            </InfoIcon>
            <base-input>
              <el-select
                v-model="item.deal_id"
                filterable
                multiple
                placeholder="Search items..."
                class="select-danger"
              >
                <el-option
                  v-for="item in allItems"
                  :key="item.id"
                  :label="`${item.name} (${item.price})`"
                  :title="item.translations"
                  :value="item.id">
                </el-option>
              </el-select>
            </base-input>
          </div>

          <base-input v-if="!isOption" label="Item image">
            <dropzone-file-upload
              ref="imageDropzone"
              v-model="imageFile"
              @addedfile="checkFile"
              :options="{
          thumbnailMethod: 'contain',
          dictDefaultMessage: 'Drop an image here to upload',
          acceptedFiles: 'image/jpeg,image/png,image/gif',
          autoProcessQueue: false,
          headers:{Authorization:'Bearer ' + jwt},
          paramName: 'image',
        }"
              :url="`${apiURL}/accounts/${currentAccountId}/menu/items/${item.id}/image`"
            >
            </dropzone-file-upload>
          </base-input>
          <p style="padding: 0; margin-top: -1.5rem; color: #32325d; font-size: 14px; font-weight: bold;" v-if="item.image">
            1 image uploaded <i style="color: #e32e2e; cursor: pointer;" class="fa fa-times-circle" @click="removeImage"></i>
          </p>

          <div>
            <label for="image-url">Image URL:</label>
            <input id="image-url" type="text" v-model="imageUrl">
            <button @click="uploadImage">Upload</button>
          </div>

          <label class="form-control-label">Beer information</label>
          <div class="row">
            <div class="col-lg-6">
              <base-input
                type="text"
                label="Untappd rating"
                disabled
                v-model="item.untappd_beer_rating">
                <small slot="prepend" class="font-weight-bold">
                  <img src="/img/icons/common/untappdrating.png"
                       style="width: 20px; height: auto;" alt="Untappd star">
                </small>
              </base-input>
            </div>
            <div class="col-lg-6">
              <base-input
                type="text"
                label="abv"
                v-model="item.abv">
                <small slot="append" class="font-weight-bold">
                  %
                </small>
              </base-input>
            </div>
          </div>

          <base-input v-if="currentRestaurant.untappd_access_token" label="Untappd beer">
            <el-select
              v-model="untappdSearch"
              filterable
              remote
              reserve-keyword
              value-key="untappd_id"
              placeholder="Search beer"
              :remote-method="searchUntappd"
              :loading="untappdSearchLoading">
              <el-option
                v-for="item in untappdResults"
                :key="item.untappd_id"
                :label="item.name + ' ⭐' + item.rating + ' (' + item.brewery + ')'"
                :value="item">
              </el-option>
            </el-select>
          </base-input>

          <base-input
            v-if="canManageAccounts"
            type="text"
            name="Custom class"
            :label="'custom_class' in item ? 'CSS custom class name (ADMIN ONLY)' : 'CSS custom class name (NEEDS BACKEND UPDATE TO ADD A PROPERTY \'custom_class\' - NOT WORKING NOW)'"
            placeholder="espresso-item"
            onkeyup="this.value = this.value.replace(/ /g, '-').replace(/[^a-zA-Z\-]/, '').toLowerCase()"
            v-model="item.custom_class"
          >
          </base-input>

        </div>

        <table v-if="mode === 'edit'" class="small pb-1 mx-auto">
          <tr>
            <td class="text-right">{{ isOption ? 'Option' : 'Item' }} created:</td>
            <td class="pl-2">{{createdAtDate}}</td>
          </tr>
          <tr>
            <td class="text-right">Last modified:</td>
            <td class="pl-2">{{modifiedAtDate}}</td>
          </tr>
        </table>

      </form>

      <div class=" col-3 py-4 px-1 border-left" style="background: #fafafa;" v-if="canManageAccounts && item.pos_id && currentRestaurant.pos_integration">
        <div v-if="posItem">
          Raw POS data (admin only)
          <BaseJson :data="posItem" size="sm" />
        </div>
        <div v-else-if="posItemMissing">
          Couldn't find POS ID {{item.pos_id}} in the POS menu.

        </div>
        <div v-else class="d-flex justify-content-center py-8">
          <div class="spinner-border text-primary" role="status">
            <span class="sr-only">Loading...</span>
          </div>
        </div>
      </div>
    </div>

    <template slot="footer">
      <base-button type="secondary" @click="onClose">{{changesMade ? 'Cancel' : 'Close' }}</base-button>
      <base-button v-if="mode === 'edit'" type="primary" @click="onsubmit">Save changes</base-button>
      <base-button v-else type="primary" @click="onsubmit">{{ isOption ? 'Create option' : 'Create product' }}
      </base-button>
    </template>
  </modal>
</template>

<script>
  import swal from "sweetalert2";
  import draggable from 'vuedraggable';
  import diffChecker from '@/mixins/diffChecker';
  import Collapse from '@/components/Collapse/Collapse';
  import ButtonRadioGroup from '@/components/ButtonRadioGroup';
  import CollapseItem from '@/components/Collapse/CollapseItem';
  import ItemTypeSelect from "@/components/Inputs/ItemTypeSelect";
  import DropzoneFileUpload from '@/components/Inputs/DropzoneFileUpload';
  import EditOptionGroupModal from "@/views/Products/EditOptionGroupModal";
  import BaseJson from "@/components/BaseJson"
  import axios from "axios";

  export default {
    mixins: [diffChecker],
    name: "EditItemModal",
    components: {
      DropzoneFileUpload,
      ButtonRadioGroup,
      EditOptionGroupModal,
      draggable,
      Collapse,
      CollapseItem,
      ItemTypeSelect,
      BaseJson
    },
    props: ["itemData", "show", "mode", "isOption", "addToParentId", "vatPercentages"],
    mounted() {
      let self = this;
      this.item = JSON.parse(JSON.stringify(this.itemTemplate));
      this.currentRestaurant.languages.forEach(function (lang, index) {
        self.item.translations.push({name: null, description: null, language: lang});
      })

      this.itemTemplate = JSON.parse(JSON.stringify(this.item))

      this.watchItemChange = false;
      if (this.itemData) {
        this.item = JSON.parse(JSON.stringify(this.itemData))
        let missingTranslations = this.itemTemplate.translations.filter(x => !self.item.translations.some(y => y.language === x.language))
        missingTranslations.forEach(function (langObject) {
          langObject.name = null;
          langObject.description = null;
          self.item.translations.push(langObject)
        })
      }

      if (this.isOption) {
        this.item.is_option = true;
      }

      //Sort the translations
      self.item.translations.sort((a, b) => self.currentRestaurant.languages.indexOf(a.language) - self.currentRestaurant.languages.indexOf(b.language))

      this.$nextTick(function () {
        this.watchItemChange = true;
      })

      this.oldItem = JSON.parse(JSON.stringify(this.item));

      if (this.vatPercentages && this.vatPercentages.length > 0) {
        this.VATOptions = [];
        this.vatPercentages.forEach((vat) => {
          console.log(vat['percentage']);
          this.VATOptions.push({
            value: vat['percentage'],
            label: `${(vat['percentage'] * 100).toFixed(0)}%`
          });
        });
      } else {
        this.VATOptions = [
          {
            value: 0.09,
            label: '9%'
          },
          {
            value: 0.21,
            label: '21%'
          }
        ];
      }

      if(this.canManageAccounts && this.item.pos_id && this.currentRestaurant.pos_integration){
        this.$store.dispatch('searchPos', this.item.pos_id)
          .then(function (res) {
            if(res.data.items && res.data.items.length){
              self.posItem = res.data.items[0];
            }else{
              self.posItemMissing = true;
            }
            self.posItemSearchCompleted = true;
          })
          .catch(function (err) {
            alert('couldn\'t retrieve pos items from the server')
            console.log(err)
            self.posItemMissing = true;
            self.posItemSearchCompleted = true;
          })
      }


    },
    watch: {
      itemData: function (newVal) {
        this.watchItemChange = false;
        this.item = JSON.parse(JSON.stringify(newVal));
        this.$nextTick(function () {
          this.watchItemChange = true;
        })
      },
      item: {
        handler(newVal) {
          if (this.watchItemChange) {
            console.log('item changed!');
            this.changesMade = true;
          }
        },
        deep: true
      },
      posSearch: function (newVal) {
        let self = this;
        if (newVal) {

          let optionGroupsArr = newVal.option_groups;
          newVal.option_groups = [];

          //Set POS item in sidebar as new value
          self.posItem = newVal;
          self.posItemMissing = false;

          //Another item than the current POS_ID was selected, and an item was selected (make sure this doesn't trigger on closing/resetting the modal)
          if (optionGroupsArr && optionGroupsArr.length > 0) {
            //item has options
            optionGroupsArr.forEach(optionGroup => {
              optionGroup.items = [];
              let options = optionGroup.options;
              delete optionGroup.options; //remove options from the object, as server can't handle this in POST

              if (!self.allOptionGroups.some(x => x.pos_id === optionGroup.pos_id)) {
                //No option group with the current POS ID exists, check the options itself first, then create group
                if (!options || !options.length) return; //Option group has no options, no need to create the group

                if (!confirm(`Do you want to create option group ${optionGroup.name}?`)) {
                  //User cancelled
                  return;
                }

                let resolveOptions = new Promise((resolve, reject) => {
                  let optionsToResolve = options.length;
                  if (optionsToResolve < 1) resolve();
                  options.forEach(option => {
                    if (!self.allItemsAndOptions.some(x => x.pos_id === option.pos_id)) {
                      //Option item or item didn't exist yet
                      delete option.created_at;
                      delete option.updated_at;
                      option.restaurant_id = this.$store.getters.currentRestaurantId;
                      option.translations = [];
                      option.active = true;
                      option.is_option = true;
                      this.$store.dispatch('newItem', option)
                        .then(function (res) {
                          //Add item ids to option group after being created
                          let items = optionGroup.items;
                          items.push(res.data.item.id);
                          optionsToResolve--;
                          if (optionsToResolve < 1) resolve();
                        })
                        .catch(function (err) {
                          alert(err)
                          optionsToResolve--;
                          if (optionsToResolve < 1) resolve();
                        })
                    } else {
                      let items = optionGroup.items;
                      //TODO REMOVE
                      console.log('option already exists, add current option group');
                      console.log(self.allItemsAndOptions.find(x => x.pos_id === option.pos_id));
                      items.push(self.allItemsAndOptions.find(x => x.pos_id === option.pos_id).id);
                      optionsToResolve--;
                      if (optionsToResolve < 1) resolve();
                    }
                  })
                });

                resolveOptions.then(() => {
                  //Create option group
                  optionGroup.restaurant_id = this.$store.getters.currentRestaurantId;
                  optionGroup.translations = [];
                  if (!(optionGroup.min_select === 1 && optionGroup.max_select === 1)) {
                    //optionGroup should be addons and not select
                    optionGroup.type = 'Addons'
                  }
                  this.$store.dispatch('newOptionGroup', optionGroup)
                    .then(function (res) {
                      //Add the newly created option group to the item.
                      let option_groups = self.item.option_groups;
                      option_groups.push(res.data.option.id);
                    })
                    .catch(function (err) {
                      alert(err)
                    })
                })

              } else {
                let option_groups = self.item.option_groups;
                //TODO REMOVE
                // console.log('option group already exists, add current option group');
                // console.log(option_groups);
                // console.log(self.allOptionGroups);
                // console.log(self.allOptionGroups.find(x => x.pos_id === optionGroup.pos_id))
                // console.log(self.allOptionGroups.find(x => x.pos_id === optionGroup.pos_id).id)
                let optionGroupId = self.allOptionGroups.find(x => x.pos_id === optionGroup.pos_id).id
                if (!option_groups.includes(optionGroupId)) option_groups.push(optionGroupId); //Only add id to option_groups if not already in the options

              }
            })
          }

          if (this.mode === 'new') {
            this.item.pos_id = newVal.pos_id;
            this.item.name = newVal.name
            this.item.price = newVal.price
            this.item.vat = newVal.vat
            this.item.untill_item_type = newVal.untill_item_type
            this.item.price_levels = newVal.price_levels;
            if (newVal.translations && newVal.translations.length) {
              newVal.translations.forEach(translation => translation.description = translation.description.replace(/<[^>]*>?/gm, ''));  //TODO remove replace when supporting rich text
              newVal.translations = [...new Map(newVal.translations.reverse().slice().reverse().map(v => [v.language, v])).values()].reverse() //Filter duplicates TODO remove when backend is fixed
              this.item.translations = newVal.translations;
            } else if (this.item.translations && newVal.description) this.item.translations[0].description = newVal.name //TODO deze geeft mogelijk problemen

            if (newVal.image !== null) {
              this.addFileFromUrl(newVal.id, newVal.image)
            }

            // if(newVal.price_levels){
            //   newVal.price_levels.forEach((priceLevel) => {
            //     self.item.price_levels.push({
            //       price_id: priceLevel.price_id,
            //       takeaway_price: priceLevel.takeaway_price,
            //       delivery_price: priceLevel.delivery_price,
            //       amount: priceLevel.amount,
            //       vat_percentage: priceLevel.vat_percentage,
            //     })
            //   })
            // }


          } else {
            if (this.item.pos_id !== newVal.pos_id && newVal.pos_id) {
              this.$refs.posIdField.$refs.input.classList.add('is-valid');
              this.$refs.posIdField.updatedByJS = true;
              this.item.pos_id = newVal.pos_id
            }
            if (this.item.name !== newVal.name && newVal.name) {
              this.$refs.autofocus.$refs.input.classList.add('is-valid');
              this.$refs.autofocus.updatedByJS = true;
              this.item.name = newVal.name
            }
            if (this.item.price !== newVal.price) {
              this.$refs.priceField.$el.querySelector("input").classList.add('is-valid');
              this.$refs.priceField.updatedByJS = true;
              this.item.price = newVal.price
            }
            if (this.item.vat !== newVal.vat && newVal.vat) {
              this.$refs.vatField.$el.querySelector(".vat-radio-button").classList.add('is-valid');
              this.$refs.vatField.updatedByJS = true;
              this.item.vat = newVal.vat
            }

            this.item.price_levels = newVal.price_levels;

            if (newVal.translations && newVal.translations.length && this.item.translations) {
              this.$refs.translations.querySelectorAll(".description").forEach(descriptionNode => descriptionNode.classList.add('is-valid'));
              this.$refs.translations.querySelectorAll(".translationName").forEach(translationNode => translationNode.querySelector("input").classList.add('is-valid'));
              for (let i = 0; i < this.item.translations.length; i++) {
                let refDescription = 'description' + i;
                let refName = 'translationName' + i;
                this.$refs[refDescription][0].updatedByJS = true;
                this.$refs[refName][0].updatedByJS = true;
              }
              newVal.translations.forEach(translation => translation.description = translation.description.replace(/<[^>]*>?/gm, ''));  //TODO remove replace when supporting rich text
              newVal.translations = [...new Map(newVal.translations.reverse().slice().reverse().map(v => [v.language, v])).values()].reverse() //Filter duplicates TODO remove when backend is fixed

              this.item.translations = newVal.translations;
            } else if (newVal.description && this.item.translations && !this.item.translations.some(translation => translation.description === newVal.description)) {
              console.log(this.$refs.description0);
              console.log(this.$refs.description0[0].$el.querySelector(".description").classList);
              console.log(newVal);
              this.$refs.description0[0].querySelector(".description").classList.add('is-valid');
              // this.$refs.description0[0].$el.querySelector(".description").classList.add('is-valid'); TODO DIT IS WSS DE JUISTE, MAAR NOG NIET FIXEN NU
              this.$refs.description0[0].updatedByJS = true;
              this.item.translations[0].description = newVal.description
            }

            // if(newVal.price_levels){
            //   newVal.price_levels.forEach((priceLevel) => {
            //     let existingPriceLevelIndex = self.item.price_levels.findIndex(values => values.price_id === priceLevel.price_id
            //       && values.takeaway_price === priceLevel.takeaway_price
            //       && values.delivery_price === priceLevel.delivery_price)
            //     console.log(`Existing price level index: ${existingPriceLevelIndex}`)
            //     if (existingPriceLevelIndex > -1) {
            //       self.item.price_levels[existingPriceLevelIndex].amount = priceLevel.amount;
            //       self.item.price_levels[existingPriceLevelIndex].vat_percentage = priceLevel.vat_percentage;
            //     } else {
            //       self.item.price_levels.push({
            //         price_id: priceLevel.price_id,
            //         takeaway_price: priceLevel.takeaway_price,
            //         delivery_price: priceLevel.delivery_price,
            //         amount: priceLevel.amount,
            //         vat_percentage: priceLevel.vat_percentage,
            //       })
            //     }
            //   })
            //   console.log(`Untill price levels: ${this.item.price_levels}`)
            // }

            this.item.untill_item_type = newVal.untill_item_type

            if (newVal.image !== null) {
              this.addFileFromUrl(newVal.id, newVal.image)
            }
          }
        }
      },
      untappdSearch: function (newVal) {
        if (newVal) {
          if(this.mode === 'new') {
            this.item.translations.forEach(translation => {
              translation.description = newVal.description;
            })
            this.item.untappd_beer_id = newVal.untappd_id;
            this.item.untappd_beer_rating = newVal.rating;
            this.item.untappd_brewery_name = newVal.original_brewery;
            this.item.abv = newVal.abv;
            this.item.tags = newVal.style.split(' - ');
          } else {
            if (newVal.description && this.item.translations) {
              this.item.translations.forEach(translation => {
                translation.description = newVal.description
              })
            }
            if (this.item.untappd_beer_id !== newVal.untappd_id && newVal.untappd_id) {
              this.item.untappd_beer_id = newVal.untappd_id
            }
            if (this.item.untappd_beer_rating !== newVal.rating && newVal.rating) {
              this.item.untappd_beer_rating = newVal.rating;
            }
            if (this.item.untappd_brewery_name !== newVal.original_brewery && newVal.original_brewery) {
              this.item.untappd_brewery_name = newVal.original_brewery;
            }
            if (this.item.abv !== newVal.abv && newVal.abv) {
              this.item.abv = newVal.abv;
            }

            let styleList = newVal.style.split(' - ');
            Array.prototype.push.apply(styleList, this.item.tags)
            this.item.tags = styleList;
          }
        }
      }
    },
    data() {
      return {
        imageUrl: '',
        imageFile: [],
        VATOptions: [
          {
            value: 0.09,
            label: '9%'
          },
          {
            value: 0.21,
            label: '21%'
          }
        ],
        watchItemChange: true,
        changesMade: false,
        dropdownVisible: false,
        itemTemplate: {
          "id": null,
          "id_code": null,
          "name": null,
          "price": null,
          "vat": null,
          "ingredients": null,
          "image": null,
          "active": true,
          "pos_id": null,
          "deal_id": [],
          "keywords": null,
          "translations": [],
          "reservation": false,
          "option_groups": [],
          "available_till": null,
          "is_option": false,
          "disabled_at": null,
          "modified_at": null,
          "created_at": null,
          "untill_item_type": null,
          "has_allergens": true,
          "allergens": [],
        },
        item: undefined,
        oldItem: undefined,
        showNewOptionGroupModal: false,
        showModalDelayed: false,
        posSearch: null,
        posSearchLoading: false,
        posSearchResults: [],
        posItem: null,
        posItemSearchCompleted: false,
        posItemMissing: false,
        untappdSearch: null,
        untappdSearchLoading: false,
        untappdResults: [],
        allergens: [
          {
            value: 'celery',
            label: 'Celery'
          }, {
            value: 'crustacean',
            label: 'Crustacean'
          }, {
            value: 'eggs',
            label: 'Eggs'
          }, {
            value: 'fish',
            label: 'Fish'
          }, {
            value: 'gluten',
            label: 'Gluten'
          }, {
            value: 'lupins',
            label: 'Lupins'
          }, {
            value: 'milk',
            label: 'Milk'
          }, {
            value: 'mustard',
            label: 'Mustard'
          }, {
            value: 'nuts',
            label: 'Nuts'
          }, {
            value: 'peanuts',
            label: 'Peanuts'
          }, {
            value: 'sesame',
            label: 'Sesame'
          }, {
            value: 'shellfish',
            label: 'Shellfish'
          }, {
            value: 'soya',
            label: 'Soya'
          }, {
            value: 'sulphite',
            label: 'Sulphite'
          }],
        showOptionGroupModalId: null,
        showOptionItemModalId: null,
        drag: false,
      };
    },
    methods: {
      async uploadImage() {
        try {
          const response = await axios.post(`${this.apiURL}/accounts/${this.currentAccountId}/menu/items/${this.item.id}/image`, {
            imageUrl: this.imageUrl,

          });
          console.log(response.data);
        } catch (error) {
          console.error(error);
        }
      },
      // bestand ophalen omzetten naar een afbeelding



      //foutmendingen
      //1. foute link
      //2. hoe kan ik die link opalen en foutmelding geven als er afbeelding in zet
      //3. foutmelding wel bestend, maar onze server zegt. Maar geen gif. Orderli server
      newOptionGroupModal(value) {
        this.showNewOptionGroupModal = value;
      },
      checkAllergens(allergenValue) {
        return !!this.item.allergens.includes(allergenValue);
      },
      changeAllergens(allergenValue) {
        if (this.item.allergens.includes(allergenValue)) {
          this.item.allergens = this.item.allergens.filter(item => item !== allergenValue);
        } else {
          this.item.allergens.push(allergenValue);
        }
      },
      addFileFromUrl(itemId, fileName) {
        let self = this;
        this.$store.dispatch('getTempImage', [itemId, fileName])
          .then(function (res) {
            // let blob = new Blob([res.data]);
            // let file = new File([blob], fileName, {
            //   type: blob.type,
            // });
            self.$refs.imageDropzone.dropzone.removeAllFiles()
            self.$refs.imageDropzone.dropzone.addFile(res.data);
          })
          .catch(function (err) {
            alert(err)
          });
      },
      removeImage() {
        this.item.image = null;
      },
      imageUpload() {
        console.log('image upload check');
        console.log(this.$refs.imageDropzone);

        if (this.$refs.imageDropzone) {
          console.log(this.$refs.imageDropzone.dropzone);
          this.$refs.imageDropzone.dropzone.processQueue();
        }
      },
      checkFile(file) {
        if (file.type !== 'image/jpeg' && file.type !== 'image/png' && file.type !== 'image/gif') {
          this.$refs.imageDropzone.dropzone.removeFile(file);
          alert(`Filetype '${file.type}' not supported. Please upload images in .jpeg, .png, or .gif format. Contact us if you need additional formats.`)
        }
      },
      onsubmit() {
        let self = this;

        if(this.currentRestaurant.pos_integration && !this.item.pos_id && !(this.addToParentId && !this.is_option && this.getCategoryFromMap(this.addToParentId).style === 'message')){
          //Item is being created or updated without a POS ID, while restaurant has a POS integration. Give a warning before continuing
          if(!confirm('You are creating a product without a POS ID! When guests order this product you will likely get an error and the order won\'t go through. Are you sure you want to create this product without a POS ID?')){
            return
          }

        }

        if (this.currentRestaurant.languages.length <= 0) {
          //TODO make a parameter if restaurants want both to be the same
          //Only one language was set, sync all changes to 'name' also to the translation name
          this.item.translations.find(x => x.language === this.currentRestaurant.languages[0]).name = this.item.name;
        }

        if (this.item.allergens.length) {
          this.item.allergens.sort();
        }

        if (this.mode === 'new') {
          //New product has been saved, do POST products call
          this.item.restaurant_id = this.$store.getters.currentRestaurantId;

          //Set the item as an option again just in case the component wasn't remounted but reused
          if (this.isOption) {
            this.item.is_option = true;
          }

          this.$store.dispatch('newItem', this.item)
            .then(function (res) {
              if (self.addToParentId) {
                if (self.isOption) {
                  let items = self.getFromOptionGroupsMap(self.addToParentId).items;
                  items.push(res.data.item.id);
                  self.$store.dispatch('updateOptionGroup', [{id: self.addToParentId, items: items}])
                } else {
                  let items = self.getCategoryFromMap(self.addToParentId).items;
                  items.push(res.data.item.id);
                  self.$store.dispatch('updateCategory', [{id: self.addToParentId, items: items}])
                }
              }
              self.item.id = res.data.item.id;
              if (self.$refs.imageDropzone) {
                self.$refs.imageDropzone.dropzone.options.url = `${self.apiURL}/accounts/${self.currentAccountId}/menu/items/${self.item.id}/image`;
                self.imageUpload();
              }
              self.$emit('createdItemId', res.data.item.id);
              self.onClose()
            })
            .catch(function (err) {
              alert(err)
            })
        } else {
          //A product was edited, do a PUT call for it
          let itemsArr = [];
          let putObj = this.getChangedKeysFromObject(this.oldItem, this.item);
          itemsArr.push(putObj);
          this.$store.dispatch('updateItem', itemsArr)
            .then(function () {
              self.imageUpload();
              self.onClose()
            })
            .catch(function (err) {
              alert(err)
            })
        }
      },
      onClose() {
        this.changesMade = false;
        this.item = JSON.parse(JSON.stringify(this.itemTemplate));
        this.posSearch = '';
        this.posSearchResults = [];
        this.$emit('modalClose');
      },
      formatDate(date) {
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
      getCategoryFromMap(id) {
        return this.categoriesMap.get(id);
      },
      getNameFromCategoryMap(id) {
        return this.categoriesMap.get(id)?.name;
      },
      getFromItemsAndOptionsMap(id) {
        return this.itemsAndOptionsMap.get(id);
      },
      getNameFromItemsAndOptionsMap(id) {
        let option = this.itemsAndOptionsMap.get(id);
        if (option) return option.name;
        return "option " + id + " not found"
      },
      getIsOptionFromItemsAndOptionsMap(id){
        return this.getFromItemsAndOptionsMap(id)?.is_option || false;
      },
      getFromOptionGroupsMap(id) {
        return this.optionGroupsMap.get(id);
      },
      getItemsFromOptionGroupsMap(id) {
        return this.optionGroupsMap.get(id)?.items;
      },
      getNameFromOptionGroupsMap(id) {
        return this.optionGroupsMap.get(id)?.name;
      },
      editOptionGroup(id, event) {
        this.showOptionGroupModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
          event.preventDefault();
        })
      },
      closeEditOptionGroupModal() {
        this.showOptionGroupModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      editOptionItem(id) {
        this.showOptionItemModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
      },
      closeEditOptionItemModal() {
        this.showOptionItemModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      searchPos(query) {
        return new Promise((resolve, reject) => {
          let self = this;
          if (query && query.length > 1) {
            this.$store.dispatch('searchPos', query)
              .then(function (res) {
                self.posSearchResults = res.data.items;
                return res.data.items;
              })
              .catch(function (err) {
                console.log(err)
                return false;
              })
          }
          return false
        })


      },
      searchUntappd(query) {
        let self = this;
        if (query && query.length > 1) {
          self.untappdSearchLoading = true;
          this.$store.dispatch('searchUntappd', query)
            .then(function (res) {
              self.untappdSearchLoading = false;
              self.untappdResults = res.data.results;
            })
            .catch(function (err) {
              alert('couldn\'t retrieve Untappd beers from the server')
              console.log(err)
            })
        }
      }
    },
    computed: {
      currentAccount() {
        return this.$store.getters.currentAccount;
      },
      createdAtDate() {
        return this.formatDate(new Date(this.item.created_at));
      },
      modifiedAtDate() {
        return this.item.modified_at ? this.formatDate(new Date(this.item.modified_at)) : 'Never';
      },
      disabledAtDate() {
        return this.formatDate(new Date(this.item.disabled_at));
      },
      allOptionGroups() {
        return this.$store.getters.optionGroupsOfCurrentRestaurant;
      },
      allItems() {
        return this.$store.getters.itemsOfCurrentRestaurant;
      },
      allItemsAndOptions() {
        return this.$store.getters.itemsAndOptionItemsOfCurrentRestaurant;
      },
      currentAccountId() {
        return this.$store.getters.currentAccountId;
      },
      currentRestaurant() {
        return this.$store.getters.currentRestaurant;
      },
      categoriesMap() {
        return this.$store.getters.categoriesMap;
      },
      optionGroupsMap() {
        return this.$store.getters.optionGroupsMap;
      },
      itemsAndOptionsMap() {
        return this.$store.getters.itemsAndOptionsMap;
      },
      jwt() {
        return localStorage.getItem('jwt');
      },
      apiURL() {
        return this.$store.state.apiURL
      },
      canManageAccounts() {
        return this.$store.getters.canManageAccounts
      },
      dragOptionsItem() {
        return {
          animation: 200,
          group: 'items',
          disabled: false,
          ghostClass: "ghost",
        };
      },
    },
  }
</script>

<style scoped>
  .allergens-active-color {
    background: #D64F28;
    color: #f6f9fc;
  }

  .allergens-color {
    background: #f6f9fc;
    color: #525f7f;
  }

  .allergens-inactive {
    cursor: default;
    opacity: 0.5;
    pointer-events: none;
  }

  .allergens-button:hover {
    transform: translateY(-1px);
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);;
  }

  .allergens-button {
    cursor: pointer;
    padding: .45rem;
    font-size: 0.75rem;
    border-radius: 0.75rem;
    text-align: center;

    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  .vat-radio-button-group {
    border-radius: .25rem;
    display: inline-block;
    overflow: hidden;
    background-color: #fff;
    background-clip: padding-box;
    border: 1px solid #dee2e6;
  }

  .vat-radio-button-group::v-deep .vat-radio-button {
    border-radius: 0;
    transition: none;
    width: 65px;
    color: #525f7f;
    font-size: 0.875rem;
    font-weight: 400;
    line-height: 1.5;
    padding: 0.625rem 0.75rem;
    background-color: unset;
  }

  .vat-radio-button-group::v-deep .vat-radio-button:first-of-type {
    border-right: 1px solid #dee2e6;
  }

  .vat-radio-button-group::v-deep .vat-radio-button:hover {
    transform: none;
  }

  div.el-table tbody td {
    border: none;
    padding: 0;
  }

  .handle {
    color: #edf0f2;
    width: 2rem;
    position: absolute;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: move;
    left: 0;
    top: 0;
  }

  .editItemModal{
    cursor: default;
  }

</style>
