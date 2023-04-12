<template>
  <div v-if="currentRestaurant && ! menuIsLoading && menuIDWasSet" ref="menusOverview">

    <EditMenuModal mode="new" :show="showNewMenuModal" v-on:modalClose="showNewMenuModal = false"/>
    <EditCardModal mode="new" :thuisCard="activeMenuIsThuis" :addToMenuId="activeMenuID" :show="showNewCardModal"
                   @updateCardPreview="$refs.menuPreview.updateCardPreview($event)"
                   v-on:modalClose="showNewCardModal = false"/>
    <EditCategoryModal mode="new" :addToCardId="activeCardID" :show="showNewCategoryModal"
                       v-on:modalClose="showNewCategoryModal = false"/>
    <EditItemModal mode="new" :addToParentId="activeCategoryID" :show="showNewItemModal"
                   :vatPercentages="currentRestaurant.vat_percentages"
                   v-on:modalClose="showNewItemModal = false"/>
    <ChangelogModal :show="showChangelogModal" v-on:modalClose="showChangelogModal = false"/>
    <PosItemsOverviewModal
      v-if="showPosOverviewModal && currentRestaurant && currentRestaurant.pos_integration &&
      currentRestaurant.pos_imported_at"
      :show="showPosOverviewModal" v-on:modalClose="showPosOverviewModal = false"
      :lastImport="formatDate(currentRestaurant.pos_imported_at)"/>
    <InactiveProductsModal :show="showInactiveItemsModal" v-on:modalClose="showInactiveItemsModal = false"/>

    <div class="container-fluid pr-0">
      <div class="row mt-3">
        <template
          v-for="menu in menusOfCurrentRestaurant"
        >
          <base-button
            @click.native.prevent="selectMenu(menu.id)"
            @dblclick.native.prevent="editMenu(menu.id)"
            :type="menu.id === activeMenuID ? 'primary' : 'secondary'"
            :key="menu.id + 'x'"
            class="pr-1"
          >
            {{menu.name}}

            <i v-if="menu.text_alert" class="fas fa-sms" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">SMS Notifcations</tippy>
            </i>
            <i v-if="menu.payment_required" class="fas fa-credit-card" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Payment Required</tippy>
            </i>
            <i v-if="menu.thuis" class="fas fa-home" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Delivery/Takeaway</tippy>
            </i>
            <i v-else-if="menu.takeaway" class="fas fa-shopping-bag" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Takeaway</tippy>
            </i>
            <i v-else-if="menu.delivery" class="fas fa-shipping-fast" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Delivery</tippy>
            </i>
            <i v-else-if="menu.takeaway_onsite" class="fas fa-walking" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">To Go Ordering</tippy>
            </i>
            <i v-else-if="menu.room_service" class="fas fa-bed" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Room service</tippy>
            </i>
            <i v-else-if="menu.menu_only" class="fas fa-eye" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">View only menu</tippy>
            </i>
            <i v-else class="fas fa-utensils" style="padding: 5px; margin: -5px 0;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Table Ordering</tippy>
            </i>
            <WeektimesInfo :times="menu.menu_times" color="inherit"  />
            <div v-if="!(menu.thuis || menu.takeaway || menu.delivery || menu.color === null)"
              :style="{'background-color': getAreaColor(menu.color)}" style="border: #F9FAFC 2px solid; border-radius: 4px; width: 17px; height: 17px; display: inline-block; margin: 0 -3px -3px 5px;">
              <tippy animation="scale" arrow arrowType="round" theme="light">Only available for QR codes in the {{menu.color}} area</tippy>
            </div>
            <el-dropdown trigger="click" class="dropdown el-dropdown-menu-button">
              <span @click.stop class="btn btn-sm btn-icon-only text-light mr-1">
                <i class="fas fa-ellipsis-v mt-2"></i>
              </span>
              <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                <el-dropdown-item v-if="menu.id !== activeMenuID" class="dropdown-item"
                                  @click.native="selectMenu(menu.id)">Select menu
                </el-dropdown-item>
                <el-dropdown-item class="dropdown-item" @click.native="editMenu(menu.id)">Edit menu</el-dropdown-item>
                <el-dropdown-item class="dropdown-item" @click.native="deleteMenu(menu.id)">Delete menu
                </el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </base-button>
          <EditMenuModal
            v-if="menu.id === showMenuModalId"
            mode="edit"
            :menuData="menu"
            :show="menu.id === showMenuModalId && showModalDelayed"
            v-on:modalClose="closeEditMenuModal"
            :key="menu.id + 'y'"
          />
        </template>

        <base-button v-if="menusOfCurrentRestaurant.length > 0" type="secondary" @click="showNewMenuModal = true">
          <b>+</b></base-button>
        <base-button v-else type="secondary" @click="showNewMenuModal = true">+ Create first menu</base-button>
        <base-button v-if="isAdmin" size="sm" type="primary" @click="showChangelogModal = true">Show recent changes
        </base-button>
        <base-button size="sm" type="primary" @click="showPosOverviewModal = true"
                     v-if="currentRestaurant && currentRestaurant.pos_integration && currentRestaurant.pos_imported_at">
          Show all POS items
        </base-button>
        <base-button size="sm" type="primary" @click="$router.push('/settings#pos')"
                     v-else-if="currentRestaurant && currentRestaurant.pos_integration">
          Run POS menu import
        </base-button>
        <base-button v-if="isAdmin" size="sm" type="primary" @click="designMode = !designMode">
          {{designMode ? 'Menueditor' : 'Menu designer (admin)'}}
        </base-button>
        <base-button class="px-3 mr-4" @click="showInactiveItemsModal = true" type="primary">
          Show inactive items
        </base-button>
        <template>
          <el-select v-model="menuLanguage" placeholder="Select Language" class="select-language" filterable>
            <el-option
              key="all"
              label="🌐 All"
              value="all"
            >
              <span style="float: left; margin-right: 13px">All</span>
              <span :key="lang.iso639" style="float: right; color: #8492a6; font-size: 13px" v-for="lang in languages">{{ lang.emoji }}</span>
            </el-option>
            <el-option
              v-for="lang in languages"
              :key="lang.iso639"
              :label="lang.emoji + ' ' + lang.iso639.toUpperCase()"
              :value="lang.iso639"
            >
              <span style="float: left; margin-right: 13px">{{ lang.name }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ lang.emoji }}</span>
            </el-option>
            <base-button type="primary" size="sm" @click="$router.push('/settings')" style="margin: 8px 8px 8px">
              Add more languages
            </base-button>
          </el-select>
        </template>
      </div>
      <div v-if="menusOfCurrentRestaurant.length > 0" class="row">
        <div class="column col-lg-8" style="padding: 0 15px;">
          <div v-show="!designMode" class="row flex-row-reverse bg-white mt-3 rounded overflow-hidden shadow">

            <div class="column column3 col-4">
              <h1 class="card-body description">Products
                <button v-if="activeCategoryID" @click="showNewItemModal = true"
                        class="btn btn-sm btn-primary float-right">+ new
                </button>
              </h1>
              <p v-if="!activeCategoryID" class="placeholder">Select a category...</p>
              <collapse>
                <draggable
                  class="list-group"
                  tag="ul"
                  v-model="items"
                  v-bind="dragOptionsItem"
                  @start="drag = true"
                  @end="drag = false"
                  draggable=".draggable-item"
                >
                  <transition-group type="transition" :name="!drag ? 'flip-list' : null">
                    <collapse-item-custom
                      :name="item.id"
                      :class="[{'active': item.id === activeItemID, 'inactive-item': !item.active && item.id !== showItemModalId}, (items.length === 1 || item.id === showItemModalId) ? 'single-item': 'draggable-item']"
                      v-for="(item, index) in items"
                      ref="itemItem"
                      :key="item.id"
                      :id="item.id"
                      @click.native.prevent="selectItem(index, item.id)"
                      @dblclick.native.prevent="editItem(item.id)"
                    >
                      <div slot="title">
                        <div class="itemCard">
                          <div v-if="items.length > 1" class="handle">
                            <i class="fa fa-grip-vertical"></i>
                          </div>
                          <div class="" style="margin-left: 2rem; display: flex; align-items: center;">
                            <div v-if="items.some(item => item.image)" class="circle-background">
                              <img v-if="item.image" class="quickSelectIcon quickselect-icon" :src="`https://orderli.com/cdn-cgi/image/width=100,quality=60/https://imagedelivery.net/xS_5nksgKmcoB2_mcBGUmA/${item.image}`" @error="$event.target.src='/img/menu/image-placeholder-350x350.png'" >
                            </div>
                            <h5 class="mb-0 px-0">
                              {{ item.name }}
                              <template v-if="itemHasActivePriceLevel(item)">
                                <tippy
                                  arrow="true"
                                  arrowType="round"
                                  animation="scale"
                                  theme="light"
                                  class="d-inline-block"
                                >
                                  <template v-slot:trigger>
                                    <span>(<span style="color: #d64f28">{{currentRestaurant.currency_icon}}{{getItemPriceLevel(item)}}</span>)</span>
                                  </template>
                                  <div>Article price of {{item.price}} is being overridden to {{currentRestaurant.currency_icon}}{{getItemPriceLevel(item)}} because of price levels</div>
                                </tippy>

                              </template>
                              <span v-else class="secundaire">({{currentRestaurant.currency_icon}}{{item.price}})</span>
                              <el-tooltip v-if="item.option_groups.length" content="Show option groups" placement="top">
                                <i @click="expandItem(item.id)" v-if="item.option_groups.length"
                                   class="far fa-clone ml-2"></i>
                              </el-tooltip>
                              <span v-for="translation in item.translations" :key="translation.id">
                                <h6 v-if="translation.language === menuLanguage || menuLanguage === 'all'" class="mb-0">
                                  <span v-if="menuLanguage === 'all'">{{languages.find(lang => translation.language === lang.iso639).emoji}}</span>
                                  {{translation.name}}
                                </h6>
                              </span>
                            </h5>
                          </div>
                          <!--                          <h5 class="mb-0">{{ item.name }}-->
                          <!--                            <span class="secundaire">(€{{ item.price }})</span>-->
                          <!--&lt;!&ndash;                            <el-tooltip v-if="item.option_groups.length" content="Show modifiers" placement="top">&ndash;&gt;-->
                          <!--&lt;!&ndash;                              <i @click="expandItem(item.id)" v-if="item.option_groups.length" class="far fa-clone ml-2"></i>&ndash;&gt;-->
                          <!--&lt;!&ndash;                            </el-tooltip>&ndash;&gt;-->
                          <!--                            <span v-for="translation in item.translations" :key="translation.id">-->
                          <!--                                <h6 v-if="translation.language === menuLanguage" class="mb-0">{{translation.name}}</h6>-->
                          <!--                            </span>-->
                          <!--                          </h5>-->
                        </div>
                        <!--                        TODO IMPORTANT change dropdown and dropdown-item to use @command instead of @click.native to prevent refresh issues-->
                        <el-dropdown trigger="click" class="dropdown el-dropdown-absolute">
                          <span @click.stop class="btn btn-sm btn-icon-only text-light mr-1">
                            <i class="fas fa-ellipsis-v mt-2"></i>
                          </span>
                          <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                            <el-dropdown-item class="dropdown-item" @click.native="editItem(item.id)">Edit item
                            </el-dropdown-item>
                            <el-dropdown-item v-if="isAdmin" class="dropdown-item" @click.native="duplicateItem(item)">Duplicate item
                            </el-dropdown-item>
                            <el-dropdown-item class="dropdown-item" @click.native="removeFromCategory(item.id)">Remove
                              from {{ getNameFromCategoriesMap(activeCategoryID) }}
                            </el-dropdown-item>
                            <!--                            <el-dropdown-item class="dropdown-item" @click.native="deleteItem(item.id)">Delete item</el-dropdown-item>-->
                          </el-dropdown-menu>
                        </el-dropdown>
                        <EditItemModal @click.native.stop="" v-if="item.id === showItemModalId" mode="edit"
                                       :itemData="item" :show="item.id === showItemModalId && showModalDelayed"
                                       :vatPercentages="currentRestaurant.vat_percentages"
                                       v-on:modalClose="closeEditItemModal"/>
                      </div>

                      <div v-if="item.option_groups.length" class="optionGroup">
                        <div v-for="optionGroup in item.option_groups" :key="optionGroup"
                             class="position-relative mt-4">
                          <div class="d-flex align-items-center">
                            <h4 class="mb-0">
                              {{ getNameFromOptionGroupsMap(optionGroup) }}
                              <span v-for="translation in getTranslationsFromOptionGroupsMap(optionGroup)" :key="translation.language">
                                <h6 v-if="translation.language === menuLanguage || menuLanguage === 'all'" class="mb-0">
                                  <span>{{languages.find(lang => translation.language === lang.iso639).emoji}}</span>
                                  {{translation.title}}
                                </h6>
                              </span>
                            </h4>



                            <el-dropdown trigger="click" class="dropdown ml-auto">
                                <span @click.stop class="btn btn-sm btn-icon-only text-light mr-1">
                                  <i class="fas fa-ellipsis-v mt-2"></i>
                                </span>
                              <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                                <el-dropdown-item class="dropdown-item" @click.native="editOptionGroup(optionGroup)">
                                  Edit option group
                                </el-dropdown-item>
                                <!--                              <el-dropdown-item class="dropdown-item" @click.native="removeFromCategory(optionGroup)">Remove from {{ item.name }}</el-dropdown-item>-->
                              </el-dropdown-menu>
                            </el-dropdown>
                          </div>
                          <EditOptionGroupModal @click.native.stop="" v-if="optionGroup === showOptionGroupModalId"
                                                mode="edit" :optionData="getFromOptionGroupsMap(optionGroup)"
                                                :show="optionGroup === showOptionGroupModalId && showModalDelayed"
                                                v-on:modalClose="closeEditOptionGroupModal"/>
                          <ul style="list-style-type: disc; padding-inline-start: 15px;">
                            <li :key="option" v-for="option in getItemsFromOptionGroupsMap(optionGroup)">
                              <div class="d-flex align-items-center justify-content-between my-2 mr-2 p-2 pl-0">
                                <h4>
                                  {{getNameFromItemsAndOptionsMap(option)}}
                                  <span v-if="getFromItemsAndOptionsMap(option) && (getFromItemsAndOptionsMap(option).price || item.option_overrides.find(obj => obj.option_item_id === option))">
                                    <span v-if="item.option_overrides.find(obj => obj.option_item_id === option)" style="color: #d64f28"> ({{item.option_overrides.find(obj => obj.option_item_id === option).price | restaurantCurrency}}) </span>
                                    <span v-else> ({{getFromItemsAndOptionsMap(option).price | restaurantCurrency}}) </span>
                                  </span>

                                  <span v-for="translation in getTranslationFromItemsAndOptionsMap(option)" :key="translation.language">
                                    <h6 v-if="translation.language === menuLanguage || menuLanguage === 'all'" class="mb-0">
                                      <span>{{languages.find(lang => translation.language === lang.iso639).emoji}}</span>
                                      {{translation.name}}
                                    </h6>
                                  </span>

                                </h4>

                                <base-button size="sm" type="secondary" @click.native="editOptionItem(option)">Edit
                                </base-button>
                              </div>
                              <EditItemModal @click.native.stop="" v-if="option === showOptionItemModalId" mode="edit"
                                             :isOption="getIsOptionFromItemsAndOptionsMap(option)"
                                             :itemData="getFromItemsAndOptionsMap(option)"
                                             :show="option === showOptionItemModalId && showModalDelayed"
                                             :vatPercentages="currentRestaurant.vat_percentages"
                                             v-on:modalClose="closeEditOptionItemModal"/>
                            </li>
                          </ul>

                          <!--                          <div class="itemCard shadow">-->
                          <!--                            <ul>-->
                          <!--                              <li :key="option" v-for="option in getFromOptionGroupsMap(optionGroup).items">-->
                          <!--                                {{getNameFromItemsAndOptionsMap(option)}}-->
                          <!--                                <base-button size="sm" type="secondary" @click="editOptionItem(option)">Edit</base-button>-->
                          <!--                                <EditItemModal @click.native.stop="" v-if="item.id === showOptionItemModalId" mode="edit"  :isOption="type === 'options'" :itemData="item" :show="item.id === showOptionGroupModalId && showModalDelayed" v-on:modalClose="closeEditOptionItemModal"/>-->
                          <!--                              </li>-->

                          <!--                            </ul>-->
                          <!--                          </div>-->

                        </div>

                        <!--                        <draggable-->
                        <!--                          class="list-group"-->
                        <!--                          tag="ul"-->
                        <!--                          v-model="item.option_groups"-->
                        <!--                          v-bind="{ 'animation': 200,-->
                        <!--                                    'group': item.id,-->
                        <!--                                    'disabled': false,-->
                        <!--                                    'ghostClass': 'ghost'}"-->
                        <!--                          @start="drag = true"-->
                        <!--                          @end="drag = false"-->
                        <!--                        >-->
                        <!--                          <transition-group type="transition" :name="!drag ? 'flip-list' : null">-->
                        <!--                            -->
                        <!--                          </transition-group>-->

                        <!--                        </draggable>-->


                      </div>
                    </collapse-item-custom>
                  </transition-group>
                </draggable>
              </collapse>

              <el-select
                v-if="activeCategoryID"
                v-model="itemSearch"
                filterable
                remote
                reserve-keyword
                value-key="id"
                v-on:change="addItem"
                placeholder="Find item to add"
                style="padding: 10px 15px; width: 100%;"
                :remote-method="searchPos"
                popper-class="h-575"
              >
                <el-option-group
                  v-for="group in itemSearchResults"
                  :key="group.label"
                  :label="group.label">
                  <el-option
                    v-for="item in group.options"
                    :key="item.id"
                    :label="item.name"
                    :value="{id:item.id,item:item, fromPos:group.itemFromPOS}">
                    <span>
                      {{item.name}} {{currentRestaurant.currency_icon}}{{item.price}} {{item.option_groups.length ? `(${item.option_groups.length} options)` : ''}} {{item.pos_id ? `[${item.pos_id}]` : ''}}
                    </span>
                  </el-option>
                </el-option-group>
              </el-select>


              <div v-if="activeCategoryID" class="text-center">
                <!--                <button class="btn" @click="showNewItemModal = true">+ Create product in {{getFromCategoriesMap(activeCategoryID).name}}</button>-->
              </div>
            </div>
            <div class="column column2 col-4">
              <h1 class="card-body description">Categories
                <button v-if="activeCardID" @click="showNewCategoryModal = true"
                        class="btn btn-sm btn-primary float-right">+ new
                </button>
              </h1>
              <p v-if="!activeCardID" class="placeholder">Select a card...</p>
              <collapse>
                <draggable
                  class="list-group"
                  tag="ul"
                  v-model="categories"
                  v-bind="dragOptionsCategory"
                  @start="drag = true"
                  @end="drag = false"
                >
                  <transition-group type="transition" :name="!drag ? 'flip-list' : null">
                    <collapse-item-custom
                      :name="category.id"
                      :class="{'active': category.id === activeCategoryID, 'inactive-item': (!category.active || !category.items.length || !checkItemActive(category.items)) && category.id !== showCategoryModalId}"
                      v-for="(category, index) in categories"
                      ref="categoryItem"
                      :key="category.id"
                      :id="category.id"
                      @click.native.prevent="selectCategory(index, category.id)"
                      @dblclick.native.prevent="editCategory(category.id)"
                    >
                      <div slot="title">
                        <div class="itemCard">
                          <div class="handle">
                            <i class="fa fa-grip-vertical"></i>
                          </div>
                          <div class="" style="margin-left: 2rem; display: flex; align-items: center;">
                            <div class="circle-background">
                              <img v-if="category.quick_select" v-bind:alt="`${category.quick_select}`" class="quickSelectIcon quickselect-icon" :src="`${serverUrl}assets/drinkIcons/ios/${category.quick_select}.png`" @error="$event.target.src='/img/menu/image-placeholder-350x350.png'" >
                            </div>
                            <h5 class="mb-0 px-0"  style="width: 65%">
                              {{ category.name }}
                              <span class="secundaire">({{category.items.length}})</span>
                              <span v-for="translation in category.translations" :key="translation.id">
                                <h6 v-if="translation.language === menuLanguage || menuLanguage === 'all'" class="mb-0">
                                  <span v-if="menuLanguage === 'all'">{{languages.find(lang => translation.language === lang.iso639).emoji}}</span>
                                  {{translation.name}}
                                </h6>
                              </span>
                            </h5>
                            <WeektimesInfo :times="category.category_times"  />
                          </div>
                          <el-dropdown trigger="click" class="dropdown el-dropdown-absolute">
                            <span @click.stop class="btn btn-sm btn-icon-only text-light mr-1">
                              <i class="fas fa-ellipsis-v mt-2"></i>
                            </span>
                            <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                              <el-dropdown-item class="dropdown-item" @click.native="editCategory(category.id)">Edit
                                category
                              </el-dropdown-item>
                              <el-dropdown-item
                                v-if="isAdmin"
                                class="dropdown-item"
                                @click.native="duplicateCategory(category)">
                                Duplicate category
                              </el-dropdown-item>
                              <el-dropdown-item class="dropdown-item" @click.native="removeFromCard(category.id)">Remove
                                from {{ getNameFromCardsMap(activeCardID) }}
                              </el-dropdown-item>
                              <!--                            <el-dropdown-item class="dropdown-item" @click.native="deleteCategory(category.id)">Delete category</el-dropdown-item>-->
                            </el-dropdown-menu>
                          </el-dropdown>
                          <EditCategoryModal @click.native.stop="" v-if="category.id === showCategoryModalId"
                                             mode="edit" :categoryData="category"
                                             :show="category.id === showCategoryModalId && showModalDelayed"
                                             v-on:modalClose="closeEditCategoryModal"/>
                        </div>
                      </div>
                      <div>
<!--                        <p>Secret developer info 🤫</p>-->
<!--                        <pre>{{category}}</pre>-->
                      </div>
                    </collapse-item-custom>
                  </transition-group>
                </draggable>
              </collapse>

              <el-select
                v-if="activeCardID"
                v-model="categorySearch"
                filterable
                value-key="id"
                v-on:change="addCategory"
                placeholder="Find category to add"
                style="padding: 10px 15px; width: 100%;"
                popper-class="h-575"
              >
                <el-option
                  v-for="category in unlinkedCategories"
                  :key="category.id"
                  :label="category.name"
                  :value="{id:category.id,category:category}">
                  <span>
                    {{category.name}} {{`(${category.items.length} items)`}}
                  </span>
                </el-option>
              </el-select>

              <div v-if="activeCardID" class="text-center">
                <!--                <button class="btn">+ Create category in {{getFromCardsMap(activeCardID).name}}</button>-->
              </div>

            </div>
            <div class="column column1 col-4">
              <h1 class="card-body description">Cards
                <button v-if="activeMenuID" @click="showNewCardModal = true" class="btn btn-sm btn-primary float-right">
                  + new
                </button>
              </h1>
              <p v-if="!activeMenuID" class="placeholder">Select a menu...</p>
              <collapse>
                <draggable
                  class="list-group"
                  tag="ul"
                  v-model="cards"
                  v-bind="dragOptions"
                  @start="drag = true"
                  @end="drag = false"
                  handle=".handle"
                >
                  <transition-group type="transition" :name="!drag ? 'flip-list' : null">
                    <collapse-item-custom
                      :name="card.id"
                      :class="{'active': card.id === activeCardID}"
                      v-for="(card, index) in cards"
                      ref="cardItem"
                      :key="card.id"
                      :id="card.id"
                      @click.native.prevent="selectCard(index, card.id)"
                      @dblclick.native.prevent="editCard(card.id)"
                    >
                      <div slot="title">
                        <div class="handle">
                          <i class="fa fa-grip-vertical"></i>
                        </div>
                        <div class="" style="display: flex; align-items: center; padding-right: 40px">
                          <h5 class="mb-0" style="max-width: 90%">
                            <div
                              :class="{'inactive-item': (!card.active || !card.categories.length || !checkCategoriesActive(card.categories) && card.id !== showCardModalId)}">
                              {{card.name}}
                              <span class="secundaire">({{card.categories.length}})</span>

                              <span v-for="translation in card.translations" :key="translation.id">
                                <h6 v-if="translation.language === menuLanguage || menuLanguage === 'all'" class="mb-0">
                                  <span v-if="menuLanguage === 'all'">{{languages.find(lang => translation.language === lang.iso639).emoji}}</span>
                                  {{translation.name}}
                                </h6>
                              </span>
                            </div>
                          </h5>
                          <div class="ml-auto">
                            <WeektimesInfo :times="card.card_times" />
                            <WeektimesInfo :times="card.timeslot_times" type="thuis" />
                          </div>
                        </div>
                        <el-dropdown trigger="click" class="dropdown el-dropdown-absolute" style="width: 12%">
                          <span class="btn btn-sm btn-icon-only text-light mr-1">
                            <i class="fas fa-ellipsis-v" style="margin-top: 0.6rem"></i>
                          </span>
                          <el-dropdown-menu class="dropdown-menu dropdown-menu-arrow show" slot="dropdown">
                            <el-dropdown-item class="dropdown-item" @click.native="editCard(card.id)">Edit card
                            </el-dropdown-item>
                            <el-dropdown-item v-if="isAdmin" class="dropdown-item" @click.native="duplicateCard(card)">Duplicate card
                            </el-dropdown-item>
                            <el-dropdown-item class="dropdown-item" @click.native="removeFromMenu(card.id)">Remove from
                              {{ getMenuNameFromMap(activeMenuID) }}
                            </el-dropdown-item>
                            <!--                            <el-dropdown-item class="dropdown-item" @click.native="deleteCard(card.id)">Delete card</el-dropdown-item>-->
                          </el-dropdown-menu>
                        </el-dropdown>
                        <EditCardModal v-if="card.id === showCardModalId" mode="edit" :thuisCard="activeMenuIsThuis"
                                       :cardData="card" :show="card.id === showCardModalId && showModalDelayed"
                                       @updateCardPreview="$refs.menuPreview.updateCardPreview($event)"
                                       v-on:modalClose="closeEditCardModal" style="cursor: default"/>

                      </div>
                      <!--                <i-->
                      <!--                  :class="card.fixed ? 'fa fa-anchor' : 'glyphicon glyphicon-pushpin'"-->
                      <!--                  aria-hidden="true"-->
                      <!--                ></i>-->
                      <div>
<!--                        <p>Secret developer info 🤫</p>-->
                        <!--                        <pre>{{card}}</pre>-->
                      </div>
                    </collapse-item-custom>
                  </transition-group>
                </draggable>
              </collapse>

              <el-select
                v-if="activeMenuID"
                v-model="cardSearch"
                filterable
                value-key="id"
                v-on:change="addCard"
                placeholder="Find card to add"
                style="padding: 10px 15px; width: 100%;"
                popper-class="h-575"
              >
                <el-option
                  v-for="card in unlinkedCards"
                  :key="card.id"
                  :label="card.name"
                  :value="{id:card.id,card:card}">
                  <span>
                    {{card.name}} {{`(${card.categories.length} categories)`}}
                  </span>
                </el-option>
              </el-select>

              <div v-if="activeMenuID" class="text-center">
                <!--                <button class="btn">+ Create card</button>-->
              </div>

            </div>
          </div>
          <card class="mt-3" v-if="designMode">
            <div slot="header">
              <h6 class="heading-small text-muted mt-3 mb-2 d-inline-block">Styling</h6>
              <InfoIcon>Make Orderli match your own branding! You can set a main color and accent color, which are used
                throughout the menu. More custom branding coming soon 👀
              </InfoIcon>
            </div>
            <div class="row mb-5">
              <div class="col-auto">
                <label class="form-control-label">
                  Brand color
                </label>
                <ColorPicker :chosenColor="restaurant.main_color" :defaultColor="'#D64F28'"
                             v-model="restaurant.main_color"/>

              </div>
              <div class="col-auto">
                <label class="form-control-label">
                  Accent color
                </label>
                <ColorPicker :chosenColor="restaurant.accent_color" :defaultColor="'#C5DEB8'"
                             v-model="restaurant.accent_color"/>
              </div>
            </div>

            <div class="row mb-5">
              <div class="col-auto">
                <CustomFontPickerGroup v-model="restaurant.fonts" />
              </div>
            </div>
            <div v-if="isAdmin" class="row mb-5">
              <div class="col-12">
                <BaseCodeEditor v-model="restaurant.custom_css" />
              </div>
            </div>
            <div slot="footer" style="float: right">
              <base-button type="secondary" native-type="submit" @click="resetComponent" :disabled="!changesMade">Cancel
                changes
              </base-button>
              <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save changes
              </base-button>
            </div>
          </card>
        </div>
        <div class="column col-lg-4 hidden-md-and-down" style="padding-left: 15px;">
          <div class="row mt-3 sticky-10" style="background: none">
            <MenuPreview ref="menuPreview" v-bind:thuis="activeMenuIsThuis" v-bind:currentMenu="currentMenu"
                         v-bind:restaurant="restaurant" v-bind:selectedMenu="activeMenuID"/>
          </div>
        </div>
      </div>

    </div>
  </div>
  <MenusSkeletonPage v-else/>
</template>

<script>
  import Vue from "vue";
  import store from "@/store";
  import swal from "sweetalert2";
  import draggable from 'vuedraggable';
  import posImport from "@/mixins/posImport";
  import formatDateTime from "@/mixins/formatDateTime";
  import Collapse from '@/components/Collapse/Collapse';
  import MenuPreview from '@/views/Products/MenuPreview';
  import ColorPicker from "@/components/Inputs/ColorPicker";
  import EditMenuModal from "@/views/Products/EditMenuModal";
  import EditCardModal from "@/views/Products/EditCardModal";
  import EditItemModal from "@/views/Products/EditItemModal";
  import InactiveProductsModal from "./InactiveProductsModal";
  import ChangelogModal from "@/views/Products/ChangelogModal";
  import EditCategoryModal from "@/views/Products/EditCategoryModal";
  import restaurantSettingsForm from '@/mixins/restaurantSettingsForm';
  import CustomFontPickerGroup from "@/components/CustomFontPickerGroup";
  import MenusSkeletonPage from '@/views/Products/MenusSkeletonPage.vue';
  import {Button, DropdownMenu, DropdownItem, Dropdown} from 'element-ui';
  import EditOptionGroupModal from "@/views/Products/EditOptionGroupModal";
  import CollapseItemCustom from '@/components/Collapse/CollapseItemCustom';
  import PosItemsOverviewModal from "@/views/Products/PosItemsOverviewModal";
  import WeektimesInfo from "@/components/WeektimesInfo";
  import BaseCodeEditor from "@/components/Inputs/BaseCodeEditor";
  import { polyfillCountryFlagEmojis } from "country-flag-emoji-polyfill";
  polyfillCountryFlagEmojis();


  export default {
    mixins: [restaurantSettingsForm, formatDateTime, posImport],
    name: "MenusOverview",
    components: {
      WeektimesInfo,
      EditOptionGroupModal,
      draggable,
      Collapse,
      CollapseItemCustom,
      EditMenuModal,
      EditCardModal,
      EditCategoryModal,
      EditItemModal,
      ChangelogModal,
      PosItemsOverviewModal,
      MenuPreview,
      MenusSkeletonPage,
      CustomFontPickerGroup,
      ColorPicker,
      [Button.name]: Button,
      [Dropdown.name]: Dropdown,
      [DropdownItem.name]: DropdownItem,
      [DropdownMenu.name]: DropdownMenu,
      InactiveProductsModal,
      BaseCodeEditor
    },
    data() {
      return {
        menuIDWasSet: false,
        activeMenuID: null,
        activeCardID: null,
        activeCategoryID: null,
        activeItemID: null,
        showNewMenuModal: false,
        showNewCardModal: false,
        showNewCategoryModal: false,
        showNewItemModal: false,
        showChangelogModal: false,
        showModalDelayed: false,
        showMenuModalId: null,
        showCardModalId: null,
        showCategoryModalId: null,
        showItemModalId: null,
        showOptionGroupModalId: null,
        showOptionItemModalId: null,
        showPosOverviewModal: false,
        drag: false,
        designMode: false,
        menuLanguage: 'en',
        itemSearch: null,
        categorySearch: null,
        cardSearch: null,
        itemSearchResults: [{
            label: "Items in Orderli",
            itemFromPOS: false,
            options: [],
          },
        ],
        showInactiveItemsModal: false,
      };
    },
    computed: {
      menuIsLoading() {
        return this.$store.getters.menuIsLoading
      },
      menusOfCurrentRestaurant() {
        return this.$store.getters.menusOfCurrentRestaurant
      },
      currentMenu() {
        return this.menusOfCurrentRestaurant.find(menu => menu.id === this.activeMenuID)
      },
      cardsOfCurrentRestaurant() {
        return this.$store.getters.cardsOfCurrentRestaurant
      },
      categoriesOfCurrentRestaurant() {
        return this.$store.getters.categoriesOfCurrentRestaurant
      },
      itemsOfCurrentRestaurant() {
        return this.$store.getters.itemsOfCurrentRestaurant
      },
      itemsNotInThisCategory() {
        return this.itemsOfCurrentRestaurant.filter(item => !this.categoriesOfCurrentRestaurant.find(category => category.id === this.activeCategoryID).items.includes(item.id));
      },
      menusMap() {
        return this.$store.getters.menusMap
      },
      cardsMap() {
        return this.$store.getters.cardsMap
      },
      categoriesMap() {
        return this.$store.getters.categoriesMap
      },
      itemsMap() {
        return this.$store.getters.itemsMap
      },
      itemsAndOptionsMap() {
        return this.$store.getters.itemsAndOptionsMap
      },
      optionGroupsMap() {
        return this.$store.getters.optionGroupsMap
      },
      currentRestaurant() {
        return this.$store.getters.currentRestaurant
      },
      languages() {
        return this.$store.state.languages.filter(language => this.restaurant?.languages.find(lang => language.iso639 === lang))
      },
      cards: {
        get() {
          return this.currentMenu?.cards.filter(card => this.getFromCardsMap(card)).map(card => this.getFromCardsMap(card));
        },
        set(value) {
          this.$store.dispatch('updateCardsOrder', {orderedArray: value, id: this.activeMenuID}); //todo move to actions later for async code
        }
      },
      categories: {
        get() {
          return this.cardsOfCurrentRestaurant.find(card => card.id === this.activeCardID)?.categories.filter(category => this.getFromCategoriesMap(category)).map(category => this.getFromCategoriesMap(category));
        },
        set(value) {
          this.$store.dispatch('updateCategoriesOrder', {orderedArray: value, id: this.activeCardID}); //todo move to actions later for async code
        }
        // let categoryIds = this.menu.card_categories.filter(mapping => mapping.card_id === this.activeCardID).map(map => map.category_id);
        // return this.menu.categories.filter(category => categoryIds.includes(category.id));
      },
      unlinkedCategories() {
        let categories = this.cardsOfCurrentRestaurant.find(card => card.id === this.activeCardID)?.categories;
        return this.categoriesOfCurrentRestaurant.filter(category => !categories.includes(category.id))
      },
      unlinkedCards(){
        let cards = this.menusOfCurrentRestaurant.find(menu => menu.id === this.activeMenuID)?.cards;
        return this.cardsOfCurrentRestaurant.filter(card => !cards.includes(card.id))
      },
      categoryHasImages() {
        return this.items.some(item => item.image)
      },
      items: {
        get() {
          return this.categoriesOfCurrentRestaurant.find(category => category.id === this.activeCategoryID)?.items.filter(item => this.getFromItemsMap(item)).map(item => this.getFromItemsMap(item));
        },
        set(value) {
          this.$store.dispatch('updateItemsOrder', {orderedArray: value, id: this.activeCategoryID}); //todo move to actions later for async code
        }
        // let itemIds = this.menu.category_items.filter(mapping => mapping.category_id === this.activeCategoryID).map(map => map.item_id);
        // return this.menu.items.filter(item => itemIds.includes(item.id));
      },
      dragOptions() {
        return {
          animation: 200,
          group: 'cards',
          disabled: false,
          ghostClass: "ghost",
        };
      },
      dragOptionsCategory() {
        return {
          animation: 200,
          group: 'categories',
          disabled: false,
          ghostClass: "ghost",
        };
      },
      dragOptionsItem() {
        return {
          animation: 200,
          group: 'items',
          disabled: false,
          ghostClass: "ghost",
        };
      },
      activeMenuIsThuis() {
        return this.menusMap?.get(this.activeMenuID)?.thuis || this.menusMap?.get(this.activeMenuID)?.takeaway || this.menusMap?.get(this.activeMenuID)?.delivery;
      },
      isAdmin() {
        return this.$store.getters.isAdmin
      },
      serverUrl() {
        return process.env.VUE_APP_SERVER_URL || 'https://web.orderli.com/'
      },
      areas() {
        return this.$store.state.areaColors
      },
    },
    methods: {
      selectMenu(id) {
        if (this.activeMenuID !== id) {
          //The click was on another card, reset selected card, category and item TODO set to select first card of new menu
          this.activeCardID = null;
          this.activeCategoryID = null;
          this.activeItemID = null;
        }
        this.activeMenuID = id;
      },
      selectCard(index, id) {
        //TODO fix if not an array, so no index and no forEach available, also for category and item
        // this.expandCard(index,id)
        if (this.activeCardID !== id) {
          //The click was on another card, reset selected category and item TODO set to select first category of new card
          this.activeCategoryID = null;
          this.activeItemID = null;
        }
        this.activeCardID = id;
        this.$refs.menuPreview.selectCard(id);
      },
      expandCard(index, id) {
        let wasActive = this.$refs.cardItem.find(item => item.id === id).active;
        this.$refs.cardItem.forEach(item => item.active = false); //TODO use for loop or other method that's faster than forEach, also for category and item
        this.$refs.cardItem.find(item => item.id === id).active = !wasActive;
      },
      selectCategory(index, id) {
        // this.expandCategory(index, id);
        if (this.activeCategoryID !== id) {
          //The click was on another category, reset selected item TODO set to select first item of new category
          this.activeItemID = null;
        }
        this.activeCategoryID = id;
        this.$refs.menuPreview.selectCategory({cardId: this.activeCardID, categoryId: id});
      },
      expandCategory(index, id) {
        let wasActive = this.$refs.categoryItem.find(item => item.id === id).active;
        this.$refs.categoryItem.forEach(item => item.active = false);
        this.$refs.categoryItem.find(item => item.id === id).active = !wasActive;
      },
      selectItem(index, id) {
        // this.expandItem(index, id);
        this.activeItemID = id;
        this.$refs.menuPreview.selectItem({cardId: this.activeCardID, categoryId: this.activeCategoryID, itemId: id});
      },
      expandItem(id) {
        let wasActive = this.$refs.itemItem.find(item => item.id === id).active;
        this.$refs.itemItem.forEach(item => item.active = false);
        this.$refs.itemItem.find(item => item.id === id).active = !wasActive;
      },
      editMenu(id) {
        this.showMenuModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
      },
      deleteMenu(id) {
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
            if (self.activeMenuID === id) {
              self.activeMenuID = null
            }
            self.$store.dispatch('deleteMenu', id)
              .then(function () {
                //TODO trigger notification of succesful deletion
              })
              .catch(function (err) {
                alert(err)
              })
          }
        })
      },
      closeEditMenuModal() {
        this.showMenuModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      editCard(id) {
        // let CardComponentClass = Vue.extend(EditCardModal)
        // let instance = new CardComponentClass({
        //   parent: this,
        //   propsData: { mode: 'edit', cardData: card, show:true }
        // })
        // instance.$mount()
        // this.$refs.menusOverview.appendChild(instance.$el);


        this.showCardModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
      },
      deleteCard(id) {
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
            if (self.activeCardID === id) {
              self.activeCardID = null
            }
            self.$store.dispatch('deleteCard', id)
              .then(function () {
                //TODO trigger notification of succesful deletion
              })
              .catch(function (err) {
                alert(err)
              })
          }
        })
      },
      duplicateCard(card){
        let self = this;
        let duplicateCard = Object.assign({}, card);
        duplicateCard.name += " copy"
        delete duplicateCard.id
        delete duplicateCard.menus

        this.$store.dispatch('newCard', duplicateCard)
          .then(function (res) {
            let id = res.data.card.id;
            let cards = self.getMenuFromMap(self.activeMenuID).cards;
            cards.splice(cards.indexOf(card.id) + 1, 0, id);
            self.$store.dispatch('updateMenu', {id: self.activeMenuID, cards: cards})
          })
          .catch(function (err) {
            alert(err)
          })
      },
      duplicateCategory(category){
        console.log(category);
        let self = this;
        let duplicateCategory = Object.assign({}, category);
        duplicateCategory.name += " copy"
        delete duplicateCategory.id
        delete duplicateCategory.cards

        this.$store.dispatch('newCategory', duplicateCategory)
          .then(function (res) {
            let id = res.data.category.id;
            let categories = self.getFromCardsMap(self.activeCardID).categories;
            categories.splice(categories.indexOf(category.id) + 1, 0, id);
            self.$store.dispatch('updateCard', {id: self.activeCardID, categories: categories})
          })
          .catch(function (err) {
            alert(err)
          })
      },
      duplicateItem(item){
        let self = this;
        let duplicateItem = Object.assign({}, item);
        duplicateItem.name += " copy"
        delete duplicateItem.id
        delete duplicateItem.categories

        this.$store.dispatch('newItem', duplicateItem)
          .then(function (res) {
            let id = res.data.item.id;
            let items = self.getFromCategoriesMap(self.activeCategoryID).items;
            items.splice(items.indexOf(item.id) + 1, 0, id);
            self.$store.dispatch('updateCategory', {id: self.activeCategoryID, items: items})
          })
          .catch(function (err) {
            alert(err)
          })
      },
      closeEditCardModal() {
        this.showCardModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      editCategory(id) {
        this.showCategoryModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
      },
      deleteCategory(id) {
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
            if (self.activeCategoryID === id) {
              self.activeCategoryID = null
            }
            self.$store.dispatch('deleteCategory', id)
              .then(function () {
                //TODO trigger notification of succesful deletion
              })
              .catch(function (err) {
                alert(err)
              })
          }
        })
      },
      closeEditCategoryModal() {
        this.showCategoryModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      editItem(id) {
        this.showItemModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
        })
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
            if (self.activeItemID === id) {
              self.activeItemID = null
            }
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
      closeEditItemModal() {
        this.showItemModalId = null;
        this.showModalDelayed = false;
        document.body.classList.remove("modal-open");
      },
      editOptionGroup(id) {
        this.showOptionGroupModalId = id;
        this.$nextTick(function () {
          this.showModalDelayed = true;
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
      removeFromMenu(cardId) {
        let newCardsArr = this.cards.filter(card => card.id !== cardId).map(card => card.id);
        this.$store.dispatch('updateMenu', {id: this.activeMenuID, cards: newCardsArr})
      },
      removeFromCard(categoryId) {
        let newCategoriesArr = this.categories.filter(category => category.id !== categoryId).map(category => category.id);
        this.$store.dispatch('updateCard', [{id: this.activeCardID, categories: newCategoriesArr}])
      },
      removeFromCategory(itemId) {
        //First filter out the itemId to remove from the category, then map the array to just the item ID's
        let newItemsArr = this.items.filter(item => item.id !== itemId).map(item => item.id);

        //Submit the changes to the store to do the PUT call and update the state
        this.$store.dispatch('updateCategory', [{id: this.activeCategoryID, items: newItemsArr}])
      },
      getMenuFromMap(id) {
        return this.menusMap.get(id);
      },
      getMenuNameFromMap(id) {
        return this.menusMap.get(id)?.name;
      },
      getFromCardsMap(id) {
        return this.cardsMap.get(id);
      },
      getNameFromCardsMap(id) {
        return this.cardsMap.get(id)?.name;
      },
      getFromCategoriesMap(id) {
        return this.categoriesMap.get(id);
      },
      getNameFromCategoriesMap(id) {
        return this.categoriesMap.get(id)?.name;
      },
      getFromItemsMap(id) {
        return this.itemsMap.get(id);
      },
      getFromItemsAndOptionsMap(id) {
        return this.itemsAndOptionsMap.get(id);
      },
      getNameFromItemsAndOptionsMap(id) {
        let option = this.itemsAndOptionsMap.get(id);
        if (option) return option.name;
        return "option " + id + " not found"
      },
      getTranslationFromItemsAndOptionsMap(id) {
        let option = this.itemsAndOptionsMap.get(id);
        if (option) return option.translations;
        return null;
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
      getTranslationsFromOptionGroupsMap(id) {
        return this.optionGroupsMap.get(id)?.translations;
      },
      getAreaColor(area){
        let areaObj = this.areas.find(areaItem => areaItem.value === area);
        return areaObj ?areaObj.color : null;
      },
      checkCategoriesActive(categories) {
        if (!categories) return false;
        let self = this;
        return categories.some(
          function (categoryID) {
            if (self.checkItemActive(self.getFromCategoriesMap(categoryID)?.items)) {
              return true
            }
          }
        )
      },
      checkItemActive(items) {
        if (!items) return false;
        let self = this;
        return items.some(
          function (itemID) {
            if (self.getFromItemsMap(itemID)?.active) {
              return true;
            }
          }
        )
      },
      searchPos(query) {
        let self = this;

        //Search local
        this.itemSearchResults[0].options = this.itemsNotInThisCategory.filter(item => {
          return item.name?.toLowerCase()
            .indexOf(query.toLowerCase()) > -1;
        });

        //Search POS
        if (query.length > 1) {
          this.$store.dispatch('searchInactiveItems', query)
            .then(function (res) {
              self.inactiveItems = res;
              console.log('Search results: ' + res)
              if (self.itemSearchResults.length < 2) {
                self.itemSearchResults.push({
                  label: 'Inactive items',
                  itemFromPOS: false,
                  options: []
                })
              }
              self.itemSearchResults[1].options = res

              if (self.currentRestaurant.pos_integration && self.currentRestaurant.pos_imported_at) {
                self.$store.dispatch('searchPos', query)
                  .then(function (res) {
                    console.log('Search results length: ' + self.itemSearchResults.length)
                    if(self.itemSearchResults.length < 3){
                      //POS results aren't included yet in the itemSearchResults, add them
                      self.itemSearchResults.push({
                        label: "Items in POS",
                        itemFromPOS: true,
                        options: [],
                      })
                    }
                    self.itemSearchResults[2].options = res.data.items;
                  })
                  .catch(function (err) {
                    console.log('couldn\'t retrieve pos items from the server')
                    console.log(err)
                  })
              }
            })
            .catch(function (err) {

            })
        }
      },
      addItem(itemObj){
        let self = this;
        if(itemObj.fromPos){
          //Item came from POS

          //Check if item does actually exist in Orderli under this POS ID, with weak comparison bc it might be string compared to int for POS ID
          if(this.allItemsAndOptions.some(item => item.pos_id == itemObj.item.pos_id) || this.inactiveItems.some(item => item.pos_id == itemObj.item.pos_id)){
            let itemInOrderli = this.allItemsAndOptions.find(item => item.pos_id == itemObj.item.pos_id) || this.inactiveItems.find(item => item.pos_id == itemObj.item.pos_id);
            let confirmAction = confirm(`There is already an item in Orderli with POS ID ${itemObj.item.pos_id} called '${itemInOrderli.name}'. Are you sure you want to import a duplicate?`);
            if (confirmAction) {
              this.importItemFromPOS(itemObj.item);
              return;
            } else {
              return;
            }
          }

          //Create a new product for this item
          this.importItemFromPOS(itemObj.item);
        } else {
          if (itemObj.item.active === false) {
            let itemsArr = [itemObj.item];
            itemsArr[0].active = true;
            this.$store.dispatch('updateInactiveItem', itemsArr)
              .then(function () {
                self.addItemToCategory(itemObj.id)
              })
              .catch(function (err) {
                alert(err)
              })
          } else {
            this.addItemToCategory(itemObj.id)
          }
        }
      },
      addCategory(categoryObj){
        let categories = this.cardsOfCurrentRestaurant.find(card => card.id === this.activeCardID)?.categories
        categories.push(categoryObj.id);
        this.category = null;
        this.$store.dispatch('updateCard', [{id: this.activeCardID, categories: categories}])
      },
      addCard(cardObj){
        let menu = this.menusOfCurrentRestaurant.find(menu => menu.id === this.activeMenuID)
        let cards = menu.cards || [];
        cards.push(cardObj.id);

        menu.cards = cards;
        this.cardSearch = null;
        this.$store.dispatch('updateMenu', menu)
      },
      importItemFromPOS(item){
        this.importItem(item, true)
          .then((res) => {
            console.log(res);
            this.addItemToCategory(res.item.id)
          })
      },
      addItemToCategory(itemId){
        let items = this.categoriesOfCurrentRestaurant.find(category => category.id === this.activeCategoryID)?.items
        items.push(itemId);
        this.itemSearch = null;
        this.$store.dispatch('updateCategory', [{id: this.activeCategoryID, items: items}])
      },
      itemHasActivePriceLevel(item){
        let returnValue = false;
        if(item.price_levels){
          item.price_levels.forEach(priceLevel => {
            if(
              ( priceLevel.price_id && priceLevel.price_id === this.currentMenu.untill_price_level_id) ||
              ( priceLevel.delivery_price && this.currentMenu.delivery) ||
              ( priceLevel.takeaway_price && (this.currentMenu.takeaway || this.currentMenu.takeaway_onsite) )
            ){
              returnValue = true
            }
          })
        }
        return returnValue
      },
      getItemPriceLevel(item){
        return item.price_levels.find(priceLevel =>
            ( priceLevel.price_id && priceLevel.price_id === this.currentMenu.untill_price_level_id ) ||
            ( priceLevel.delivery_price && this.currentMenu.delivery) ||
            ( priceLevel.takeaway_price && (this.currentMenu.takeaway || this.currentMenu.takeaway_onsite) )
          ).amount;

      },
    },
    mounted() {
      let self = this;
      if (this.menuIsLoading) {
        let watcher = store.watch(() => store.getters.menuIsLoading, menuIsLoading => {
          watcher();
          if (! menuIsLoading) {
            this.activeMenuID = this.menusOfCurrentRestaurant[0]?.id;
            this.activeCardID = this.menusOfCurrentRestaurant[0]?.cards[0];
            this.activeCategoryID = this.cardsOfCurrentRestaurant.find(card => card.id === this.activeCardID)?.categories[0];
            this.activeItemID = this.categoriesOfCurrentRestaurant.find(category => category.id === this.activeCategoryID)?.items[0];
            if(this.languages && this.languages.length) this.menuLanguage = this.languages[0].iso639
            self.menuIDWasSet = true;
          }
        })
      } else {
        setTimeout(() => {
          self.activeMenuID = self.menusOfCurrentRestaurant[0]?.id;
          if(this.languages && this.languages.length) this.menuLanguage = this.languages[0].iso639
          self.menuIDWasSet = true;
        }, 10)
      }
    },
    created() {
      this.$store.dispatch('getMenuIfUndefined');
    }
  }
</script>

<style>
  .vertical.nav-pills .nav-item:not(:last-child) {
    padding-right: 0;
  }

  .button {
    margin-top: 35px;
  }

  .flip-list-move {
    transition: transform 0.5s;
  }

  .no-move {
    transition: transform 0s;
  }

  .ghost {
    opacity: 0.5;
    background: #c8ebfb;
  }

  .list-group {
    min-height: 20px;
  }

  .list-group-item {
    cursor: move;
  }

  .list-group-item i {
    cursor: pointer;
  }

  .list-group .header {
    background: #fff;
  }

  .list-group .active .header, .list-group .active .header h5 {
    /*background: #d64f2810;*/
    background: #F6F9FC50;
    color: #d64f28;
  }

  h1.card-body {
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
    margin-bottom: 0;
    background: #FBFAF8;
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
  }

  .placeholder {
    padding: 1.25rem 1.5rem;
  }

  .column {
    padding: 0;
    /*TODO reduce negative spread with visability hidden top and bottm*/
  }

  .column1, .column2 {
    background: white;
    box-shadow: 1px 0 6px 0 #00000015;
    /*z-index: 1;*/
  }

  .column1 {
    /*z-index: 2;*/
  }

</style>

<style scoped>
  .btn:hover, .btn:focus, .btn:not(:disabled):not(.disabled):active:focus, .btn:not(:disabled):not(.disabled).active:focus {
    transform: none;
    box-shadow: none;
  }

  .btn.btn-secondary:hover, .btn.btn-primary:hover {
    box-shadow: 0 7px 14px rgba(50, 50, 93, 0.1), 0 3px 6px rgba(0, 0, 0, 0.08);
  }

  .btn:not(.btn-primary):hover {
    color: #000
  }

  .sticky-10 {
    position: sticky;
    top: 10px;
  }

  .inactive-item {
    opacity: .3;
  }

  .select-language /deep/ .el-input__inner {
    height: 54px;
    width: 94px;
    box-shadow: 0 4px 6px rgb(50 50 93 / 11%), 0 1px 3px rgb(0 0 0 / 8%);
  }

  .btn-primary .el-dropdown-menu-button .btn {
    color: white !important;
  }

  .el-dropdown-absolute {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    right: 0;
  }

  .el-dropdown-menu__item {
    line-height: unset;
  }

  .el-dropdown-menu__item:focus, .el-dropdown-menu__item:not(.is-disabled):hover {
    color: #16181b;
    background-color: #f6f9fc;
  }

  .circle-background {
    height: 30px;
    background: #eceff1;
    border-radius: 50%;
    -webkit-box-shadow: 1px 1px 5px hsl(0deg 0% 69% / 23%);
    box-shadow: 1px 1px 5px hsl(0deg 0% 69% / 23%);
    width: 30px;
    border: none;
    margin-left: -3px;
    margin-right: 8px;
    padding: 0;
    display: -webkit-inline-box;
    display: -ms-inline-flexbox;
    display: inline-flex;
    vertical-align: middle;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
  }

  .quickSelectIcon {
    max-width: 15px;
    margin: 0 auto;
    filter: invert(15%);
  }

  .secundaire {
    color: #909fb0;
  }

  .optionGroup h5 {
    padding: 1.25rem 1.5rem 1.25rem 2rem;
  }

  .optionGroup {
    margin-top: -1.5rem;
  }

  .optionGroup .itemCard {
    margin-bottom: 5px;
    border-radius: 10px;
    border: 1px solid #bbbbbb;
  }

</style>
