<template>
  <div class="">
    <div v-if="currentRestaurant && restaurant && currentAccount">
      <card headerClasses="settingsCardHeader">
        <template slot="header">
          <span class="mr-1" style="font-weight: bold">Restaurant</span>
          <div class="position-sticky d-flex top-3 ml-auto" style="z-index: 900">
            <base-button
              size="sm"
              class="ml-auto"
              type="secondary"
              native-type="submit"
              @click="resetComponent"
              :disabled="!changesMade"
              >Cancel changes
            </base-button>
            <base-button
              size="sm"
              type="primary"
              native-type="submit"
              @click.native="submitWithValidation"
              :disabled="!changesMade"
              >Save changes
            </base-button>
          </div>
        </template>
        <div class="maxWidthBody">
          <validation-observer v-slot="{ handleSubmit }" ref="formValidator">
            <form ref="form" @submit.prevent="handleSubmit(onSubmit)">
              <div class="row">
                <div class="col-6">
                  <base-input
                    type="text"
                    label="Restaurant name"
                    placeholder="Restaurant Name"
                    v-model="restaurant.name"
                  >
                    <InfoIcon slot="appendLabel"
                      >This is the name guests will see on the menu when mentioning your
                      business.
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6">
                  <base-input
                    type="text"
                    label="Database name"
                    :disabled="!isAdmin"
                    v-model="restaurant.database_name"
                    @keydown="nameKeydown($event)"
                  >
                    <template slot="appendLabel">
                      <InfoIcon interactive>
                        This is the internal database name of your location. This also
                        determines your takeaway / delivery link to be
                        <a href="`https://thuis.orderli.com/${restaurant.database_name}`"
                          >https://thuis.orderli.com/{{ restaurant.database_name }}</a
                        >
                      </InfoIcon>
                      <badge v-if="isAdmin" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                    </template>
                  </base-input>
                </div>
                <template v-if="isAdmin">
                  <div class="col-6">
                    <base-input
                      label="Last billed at"
                      prepend-icon="ni ni-calendar-grid-58"
                    >
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <flat-picker
                        slot-scope="{ focus, blur }"
                        @on-open="focus"
                        @on-close="blur"
                        :config="{ allowInput: true }"
                        class="form-control datepicker"
                        v-model="restaurant.last_billed_at"
                      >
                      </flat-picker>
                    </base-input>
                  </div>
                  <div class="col-6">
                    <base-input
                      label="Payments last billed at"
                      prepend-icon="ni ni-calendar-grid-58"
                    >
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <flat-picker
                        slot-scope="{ focus, blur }"
                        @on-open="focus"
                        @on-close="blur"
                        :config="{ allowInput: true }"
                        class="form-control datepicker"
                        disabled
                        v-model="restaurant.payments_last_billed_at"
                      >
                      </flat-picker>
                    </base-input>
                  </div>
                  <div class="col-6">
                    <base-input
                      label="Trial end date"
                      prepend-icon="ni ni-calendar-grid-58"
                    >
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <flat-picker
                        slot-scope="{ focus, blur }"
                        @on-open="focus"
                        @on-close="blur"
                        :config="{ allowInput: true }"
                        class="form-control datepicker"
                        v-model="restaurant.trial_ends_at"
                      >
                      </flat-picker>
                    </base-input>
                  </div>
                  <div v-if="restaurant.stripe_subscription_id === null" class="col-6">
                    <label class="form-control-label">Stripe subscription</label>
                    <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                      >Admin</badge
                    >
                    <base-button
                      style="background-color: #635bff; color: white; border: none"
                      @click="createStripeSubscription"
                    >
                      <span
                        ><img
                          src="img/icons/common/stripe.svg"
                          style="width: 20px; height: auto; margin-right: 1rem"
                          alt="Stripe logo"
                      /></span>
                      <span>Create Stripe subscription</span>
                    </base-button>
                  </div>
                  <div class="col-6">
                    <base-input label="Hubspot customer">
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <el-select
                        v-model="restaurant.hubspot_company_id"
                        filterable
                        remote
                        reserve-keyword
                        placeholder="Search for restaurant in Hubspot"
                        :remote-method="searchHubspot"
                        :loading="hubspotLoading"
                      >
                        <el-option
                          v-for="result in hubspotResults"
                          :key="result.id"
                          :label="
                            result.properties.name + ' (' + result.properties.domain + ')'
                          "
                          :value="result.id"
                        >
                        </el-option>
                      </el-select>
                    </base-input>
                  </div>
                  <div class="col-6">
                    <base-input label="Hubspot deal">
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <el-select
                        v-model="restaurant.hubspot_deal_id"
                        filterable
                        remote
                        reserve-keyword
                        placeholder="Connect the hubspot deal to customer"
                        :remote-method="searchHubspotDeals"
                        :loading="hubspotDealsLoading"
                      >
                        <el-option
                          v-for="result in hubspotDealsResults"
                          :key="result.id"
                          :label="result.properties.dealname"
                          :value="result.id"
                        >
                        </el-option>
                      </el-select>
                    </base-input>
                  </div>
                  <div class="col-6">
                    <base-input label="Moneybird contact">
                      <badge slot="appendLabel" rounded type="secondary" class="ml-2"
                        >Admin</badge
                      >
                      <el-select
                        v-model="restaurant.moneybird_contact_id"
                        filterable
                        remote
                        reserve-keyword
                        placeholder="Search for a moneybird contact"
                        :remote-method="searchMoneybird"
                        :loading="moneybirdLoading"
                      >
                        <el-option
                          v-for="result in moneybirdResults"
                          :key="result.id"
                          :label="
                            result.company_name +
                            ' (' +
                            result.address1 +
                            ' ' +
                            result.city +
                            ')'
                          "
                          :value="result.id"
                        >
                        </el-option>
                      </el-select>
                    </base-input>
                  </div>
                </template>

                <div v-if="restaurant.logo" class="col-6">
                  <img
                    :alt="restaurant.logo"
                    style="max-height: 100px; max-width: 100%"
                    :src="
                      'https://imagedelivery.net/xS_5nksgKmcoB2_mcBGUmA/' +
                      restaurant.logo +
                      '/public'
                    "
                    @error="placeholderImgUrl"
                  />
                </div>
                <div :class="restaurant.logo ? 'col-6' : 'col-12'">
                  <dropzone-file-upload
                    ref="imageDropzone"
                    v-model="imageFile"
                    @addedfile="checkFile"
                    :options="{
                      thumbnailMethod: 'contain',
                      dictDefaultMessage: restaurant.logo
                        ? 'Drop a new logo here to change the current logo'
                        : 'Drop a logo here to upload',
                      acceptedFiles: 'image/jpeg,image/png,image/gif',
                      headers: { Authorization: 'Bearer ' + jwt },
                      paramName: 'logo',
                      maxFilesize: 0.2, //MB
                    }"
                    :url="`${apiURL}/accounts/${currentAccountId}/restaurants/${currentRestaurantId}/logo`"
                  >
                    <!--                  <img :alt="restaurant.logo" :src="$store.state.logoURL + restaurant.database_name + '/' + restaurant.logo" @error="placeholderImgUrl">-->
                  </dropzone-file-upload>
                </div>
              </div>

              <div class="row">
                <div class="col-12 col-lg-6 mb-4">
                  <h6 class="heading-small text-muted mb-2 d-inline-block">
                    Opening times
                  </h6>
                  <InfoIcon>
                    You can configure the opening times of your business. Right now the
                    opening times are used to limit the available timeslots for takeaway
                    and delivery orders. More granular control for takeaway and delivery
                    timeslots and menu visibility can be found in the time settings of
                    menus and cards in the menu editor.
                  </InfoIcon>
                  <WeektimesTable
                    v-model="restaurant.opening_times"
                    name="opening"
                    startProp="opening_at"
                    startName="Opening at"
                    endProp="closing_at"
                    endName="Closing at"
                  />
                </div>
                <div class="col-6">
                  <h6 class="heading-small text-muted mb-2 d-inline-block">
                    VAT percentages
                  </h6>
                  <InfoIcon> Configure the VAT percentages. </InfoIcon>
                  <EditVATPercentages
                    v-model="restaurant.vat_percentages"
                    rules="required"
                  />
                </div>
                <div
                  class="col-6"
                  v-if="
                    restaurant.pos_integration === 'untill' ||
                    restaurant.pos_integration === 'lightspeed_k'
                  "
                >
                  <h6 class="heading-small text-muted mb-2 d-inline-block">Courses</h6>
                  <InfoIcon> Configure the courses. </InfoIcon>
                  <EditCourses v-model="restaurant.courses" rules="required" />
                </div>
              </div>

              <!-- Custom midnight time -->
              <div class="row mt-4">
                <div class="col-12 col-6 col-lg-3">
                  <label class="form-control-label">
                    Custom midnight time
                    <InfoIcon>
                      By configuring a custom midnight time, revenue made in a new day but before restaurant closing time can be included in the revenue of previous day. This improves analytics accuracy.
                    </InfoIcon>
                  </label>
                  <base-input :value="formatToTimestring(restaurant.custom_midnight_time)" v-on:inputevt="formatToSecondsTimeslot($event)" name="custom_midnight_time" type="time"/>
                </div>
              </div>

              <!-- Address -->
              <h6 class="heading-small text-muted mb-2">Address</h6>
              <div class="row mb-4">
                <div class="col-6 col-lg-5">
                  <base-input
                    type="text"
                    label="Address"
                    placeholder="Turfmarkt 11"
                    v-model="restaurant.address"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="City"
                    placeholder="Leeuwarden"
                    v-model="restaurant.city"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-3">
                  <base-input
                    label="Postal Code"
                    placeholder="1111AA"
                    v-model="restaurant.postal_code"
                  >
                  </base-input>
                </div>
                <div class="col-6">
                  <base-input
                    type="text"
                    label="Country"
                    placeholder="Country"
                    v-model="restaurant.country"
                  >
                  </base-input>
                </div>
                <div class="col-6">
                  <base-input label="Timezone">
                    <el-select
                      v-model="restaurant.timezone"
                      filterable
                      class="select-danger"
                    >
                      <el-option
                        v-for="timezone in timezones"
                        :key="timezone.name"
                        :label="timezone.rawFormat"
                        :value="timezone.name"
                      >
                      </el-option>
                    </el-select>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="VAT Number"
                    placeholder="VAT Number"
                    v-model="restaurant.vat_number"
                  >
                  </base-input>
                </div>
              </div>

              <span style="font-weight: bold">Menu Settings</span>
              <div class="row mt-2">
                <div class="col-12 col-lg-7">
                  <base-input label="Menu languages">
                    <InfoIcon slot="appendLabel"
                      >Orderli supports any language in the world. Add the languages you
                      want your menu to be available in, after saving you can enter each
                      new translation to every item, category, card, and option in the
                      menu editor.
                    </InfoIcon>
                    <el-select
                      v-model="restaurant.languages"
                      filterable
                      multiple
                      class="select-danger"
                    >
                      <el-option
                        v-for="language in languages"
                        :key="language.iso639"
                        :label="language.emoji + ' ' + language.name"
                        :value="language.iso639"
                      >
                        <span style="float: left">{{
                          language.emoji + " " + language.name
                        }}</span>
                        <span style="float: right; color: #8492a6; font-size: 13px">{{
                          language.native
                        }}</span>
                      </el-option>
                    </el-select>
                  </base-input>
                </div>

                <div class="col-6 col-lg-2">
                  <base-input
                    type="text"
                    label="Currency"
                    placeholder="Currency"
                    v-model="restaurant.currency"
                  >
                    <InfoIcon slot="appendLabel"
                      >Please use the ISO 3-letter code to set your currency. E.g.: EUR or
                      USD
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-3">
                  <base-input
                    type="text"
                    label="Currency icon"
                    v-model="restaurant.currency_icon"
                  >
                    <InfoIcon slot="appendLabel"
                      >Configure any character, icon, emoji, or text that you want to
                      display in front of the price-number on the menu. Default is €
                    </InfoIcon>
                  </base-input>
                </div>
              </div>

              <div class="row">
                <div class="col-12">
                  <base-input type="text" label="Order message">
                    <textarea
                      rows="1"
                      class="form-control"
                      v-model.lazy="restaurant.order_message"
                    ></textarea>
                  </base-input>
                </div>
                <div class="col-12">
                  <base-input label="Services (only for table ordering)">
                    <el-select
                      v-model="restaurant.services"
                      multiple
                      class="select-danger"
                    >
                      <el-option
                        v-for="service in availableServices"
                        :key="service.value"
                        :label="service.label"
                        :value="service.value"
                      >
                        <span>
                          <InfoIcon
                            class="mr-2"
                            placement="left"
                            :flipDuration="0"
                            :popperOptions="{
                              modifiers: {
                                preventOverflow: {
                                  enabled: false,
                                },
                                hide: {
                                  enabled: false,
                                },
                              },
                            }"
                            >{{ service.description }}</InfoIcon
                          >
                          {{ service.label }}</span
                        >
                      </el-option>
                    </el-select>
                  </base-input>
                </div>
                <div class="col-12">
                  <base-input label="Prevent guests from leaving comments on items">
                    <InfoIcon slot="appendLabel"
                      >Don't want guests to be able to leave comments on items or on their
                      orders? Switch the comment boxes on the menu off by activating this
                      switch.
                    </InfoIcon>
                    <base-switch
                      class="mr-1"
                      on-text="On"
                      off-text="Off"
                      v-model="restaurant.hide_commentbox"
                    ></base-switch>
                  </base-input>
                </div>
              </div>

              <h6 class="heading-small text-muted mt-3 mb-2 d-inline-block">
                <i
                  class="fas fa-lock"
                  v-if="
                    currentAccount.subscription === null ||
                    !currentAccount.subscription.features.includes('custom_colors')
                  "
                ></i>
                Styling
                <badge
                  slot="appendLabel"
                  rounded
                  type="secondary"
                  class="ml-2"
                  v-if="
                    currentAccount.subscription === null ||
                    !currentAccount.subscription.features.includes('custom_colors')
                  "
                  >STANDARD</badge
                >
              </h6>
              <InfoIcon
                >Make Orderli match your own branding! You can set a main color and accent
                color, which are used throughout the menu. More custom branding coming
                soon 👀
              </InfoIcon>
              <div class="row">
                <div class="col-auto">
                  <label class="form-control-label"> Brand color </label>
                  <ColorPicker
                    :chosenColor="restaurant.main_color"
                    :defaultColor="'#D64F28'"
                    v-model="restaurant.main_color"
                    :disabled="
                      currentAccount.subscription === null ||
                      !currentAccount.subscription.features.includes('custom_colors')
                    "
                  />
                </div>
                <div class="col-auto">
                  <label class="form-control-label"> Accent color </label>
                  <ColorPicker
                    :chosenColor="restaurant.accent_color"
                    :defaultColor="'#C5DEB8'"
                    v-model="restaurant.accent_color"
                    :disabled="
                      currentAccount.subscription === null ||
                      !currentAccount.subscription.features.includes('custom_colors')
                    "
                  />
                </div>
              </div>
              <div class="row mb-5">
                <div class="col-6">
                  <!--                <base-input label="Custom font headings">-->
                  <!--                  <font-picker class="fontPicker" api-key="AIzaSyCjfHaKIeVqhNrbQQFAX8hEeg5lO-lLjSo" :options="{sort: 'popularity'}" :active-font="fontFamilyHeading" @change="fontChosenHeading"></font-picker>-->
                  <!--                </base-input>-->

                  <!--                <base-input label="Custom font other text">-->
                  <!--                  <font-picker class="fontPicker" api-key="AIzaSyCjfHaKIeVqhNrbQQFAX8hEeg5lO-lLjSo" :options="{sort: 'popularity'}" :active-font="fontFamilyMain" @change="fontChosenMain"></font-picker>-->
                  <!--                </base-input>-->

                  <!--                TODO font variants-->
                  <!--                <base-input-->
                  <!--                  label="Timezone"-->
                  <!--                  v-if="font && font.variants.length"-->
                  <!--                >-->
                  <!--                  <el-select-->
                  <!--                    v-model="fontVariant"-->
                  <!--                    filterable-->
                  <!--                    class="select-danger"-->
                  <!--                  >-->
                  <!--                    <el-option-->
                  <!--                      v-for="variant in font.variants"-->
                  <!--                      :key="variant"-->
                  <!--                      :label="variant"-->
                  <!--                      :value="variant">-->
                  <!--                    </el-option>-->
                  <!--                  </el-select>-->
                  <!--                </base-input>-->

                  <!--                {{font}}-->
                </div>
              </div>

              <span class="mr-1" style="font-weight: bold">Export Settings</span>
              <div class="row mt-2">
                <div class="col-6">
                  <base-input
                    inputClasses="input-group"
                    type="email"
                    autocomplete="email"
                    placeholder="youremail@mail.com"
                    label="Automatically email orders data daily"
                    v-model="restaurant.auto_export_recipient"
                    :disabled="!restaurant.auto_export"
                    class="customPadding"
                  >
                    <template v-slot:prepend>
                      <base-switch
                        class="mr-1"
                        on-text="On"
                        off-text="Off"
                        v-model="restaurant.auto_export"
                      ></base-switch>
                    </template>
                  </base-input>
                </div>

                <div class="col-6">
                  <label class="form-control-label">Item type settings</label>
                  <base-button
                    v-if="!configureItemTypes"
                    class="d-block"
                    type="secondary"
                    @click="
                      $store.dispatch('getItemTypesIfUndefined');
                      configureItemTypes = true;
                    "
                    >Configure
                  </base-button>
                  <div v-if="configureItemTypes">
                    <div :key="type.id" v-for="type in itemTypes">
                      <base-input group class="mb-3">
                        <div class="input-group">
                          <input
                            v-model="type.name"
                            type="text"
                            class="form-control"
                            placeholder="Item type name"
                            aria-label="Item type name"
                            aria-describedby="button-addon3"
                          />
                          <div class="input-group-append">
                            <button
                              class="btn btn-outline-primary"
                              type="button"
                              id="button-addon2"
                              @click="updateItemType(type)"
                            >
                              Save
                            </button>
                          </div>
                        </div>
                      </base-input>
                    </div>
                    <base-input group class="mb-3">
                      <div class="input-group">
                        <input
                          v-model="createItemTypeInput"
                          :disabled="creatingItemType"
                          type="text"
                          class="form-control"
                          placeholder="Item type name"
                          aria-label="Item type input"
                          aria-describedby="button-addon3"
                        />
                        <div class="input-group-append">
                          <button
                            class="btn btn-outline-primary"
                            type="button"
                            id="button-addon3"
                            :disabled="!createItemTypeInput || creatingItemType"
                            @click="createItemType"
                          >
                            Create new
                          </button>
                        </div>
                      </div>
                    </base-input>
                  </div>
                </div>
              </div>

              <span class="mr-1" style="font-weight: bold">Untappd settings</span>
              <div class="row mt-2">
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Untappd API key"
                    placeholder="abcdefghij"
                    v-model="restaurant.untappd_access_token"
                  >
                    <InfoIcon info slot="appendLabel">
                      Can be found in the settings of Untappd Business
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Untappd email address"
                    placeholder="example@email.com"
                    v-model="restaurant.untappd_email_address"
                  >
                    <InfoIcon info slot="appendLabel">
                      Email address used for the Untappd Business account
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Untappd venue ID"
                    placeholder="12345678"
                    v-model="restaurant.untappd_venue_id"
                  >
                    <InfoIcon info slot="appendLabel"
                      >The venue ID is in the URL of Untappd business:
                      <pre>business.untappd.com/venues/<b>1234</b></pre>
                    </InfoIcon>
                  </base-input>
                </div>
              </div>

              <span class="mr-1" style="font-weight: bold">Google review settings</span
              ><br />
              <span class="mr-1" style="font-size: 14px"
                >The Google Reviews URL can be found by following
                <a
                  href="https://support.google.com/business/answer/7035772?hl=nl"
                  target="_blank"
                  >this</a
                >
                article.</span
              >
              <div class="row mt-2">
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Google Review URL"
                    placeholder="https://g.page/r/CcZSov-EybPDEAg/review"
                    v-model="restaurant.google_reviews_url"
                  >
                    <InfoIcon info slot="appendLabel">
                      Follow the above tutorial to find the URL.
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Review question"
                    :placeholder="`Wat vind je van ${currentRestaurant.name}?`"
                    v-model="restaurant.review_question"
                  >
                    <InfoIcon info slot="appendLabel">
                      The first question the guest sees.
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Negative review"
                    placeholder="Kan beter"
                    v-model="restaurant.review_negative"
                  >
                    <InfoIcon info slot="appendLabel"> Negative button text </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Positive review"
                    placeholder="Geweldig"
                    v-model="restaurant.review_positive"
                  >
                    <InfoIcon info slot="appendLabel"> Positive button text </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Google review question"
                    placeholder="Laat je review achter op Google Reviews!"
                    v-model="restaurant.google_review_question"
                  >
                    <InfoIcon info slot="appendLabel">
                      The question asked after the guests clicks on the positive button
                    </InfoIcon>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Feedback question"
                    placeholder="Wat kunnen we het de volgende keer beter doen?"
                    v-model="restaurant.feedback_question"
                  >
                    <InfoIcon info slot="appendLabel">
                      The question asked after the guests clicks on the negative button
                    </InfoIcon>
                  </base-input>
                </div>
              </div>

              <span class="mr-1" style="font-weight: bold">Mailchimp settings</span>
              <div class="row mt-2">
                <div class="col-12 col-lg-4 mb-3">
                  <base-button
                    style="background-color: #fbeeca; color: black; border: none"
                    @click="authenticateMailchimp"
                  >
                    <span
                      ><img
                        src="img/icons/common/mailchimp.png"
                        style="width: 20px; height: auto; margin-right: 1rem"
                        alt="Mailchimp logo"
                    /></span>
                    <span>{{
                      restaurant.mailchimp_authenticated
                        ? "Reconnect Mailchimp"
                        : "Connect Mailchimp"
                    }}</span>
                  </base-button>
                </div>
              </div>

              <span class="mr-1" style="font-weight: bold">Other Settings</span>
              <div class="row mt-2">
                <div class="col-6 col-lg-4">
                  <base-input
                    type="number"
                    label="Menu editor PIN"
                    placeholder="1234"
                    v-model="restaurant.menu_editor_pin"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Google Analytics code"
                    placeholder="UA-XXXXXXXX"
                    v-model="restaurant.gtag"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Facebook Pixel"
                    placeholder="12345678987654321"
                    v-model="restaurant.facebook_pixel"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Facebook Access Token"
                    placeholder="12345678987654321"
                    v-model="restaurant.facebook_access_token"
                  >
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="Gifty API Key"
                    placeholder="12345678987654321"
                    v-model="restaurant.gifty_api_key"
                  >
                  </base-input>
                </div>
                <div class="col-12 col-md-6 col-lg-4">
                  <base-input
                    type="text"
                    :label="`Custom hostname`"
                    placeholder="web.orderli.com"
                    :disabled="currentAccount.subscription === null || !currentAccount.subscription.features.includes('custom_domain')"
                    v-model="restaurant.custom_hostname">
                    <i v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('custom_domain')" class="fas fa-lock" slot="prependLabel" style="font-size: 12px"></i>
                    <badge v-if="currentAccount.subscription === null || !currentAccount.subscription.features.includes('custom_domain')" slot="appendLabel" rounded type="secondary" class="ml-2">PRO</badge>
                    <small v-if="customDomainStatus === 'active'" slot="append" style="color: green;" title="Hostname successfully added">
                      <i class="fas fa-check-circle"></i>
                    </small>
                    <small v-else-if="customDomainStatus === 'pending'" slot="append" style="color: #555555;" title="Hostname pending">
                      <i class="fas fa-spinner"></i>
                    </small>
                    <small v-else slot="append" style="color: #555555;" title="Status unknown">
                      <i class="fas fa-question-circle"></i>
                    </small>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input
                    type="text"
                    label="New orders notification email"
                    placeholder="email@example.com"
                    v-model="restaurant.notify_new_orders_email"
                  >
                  </base-input>
                </div>
                <!--              <pre>{{resrestauranttaurant}}</pre>-->

                <!--              TODO-->
                <!--              thuis_autoswipe_timing-->
                <!--              GET/POST/PUT item types-->
                <!--              "contact_email": null,-->
                <!--              "closed": false,-->
                <!--              "closed_hours": 0,-->
                <!--              "closed_reason": null,-->
                <!--              "closed_at": "Mon, 13 Sep 2021 11:35:16 GMT"-->

                <div
                  v-if="restaurant.pos_integration === 'lightspeed'"
                  class="col-6 col-lg-4"
                >
                  <base-input label="Send customer data to Lightspeed">
                    <InfoIcon info slot="appendLabel"
                      >Send guest data from orders to Lightspeed POS to save them as
                      customers.
                    </InfoIcon>
                    <base-switch
                      class="mr-1"
                      on-text="On"
                      off-text="Off"
                      v-model="restaurant.lightspeed_send_customer_data"
                    ></base-switch>
                  </base-input>
                </div>
                <div
                  v-if="
                    restaurant.pos_integration === null ||
                    restaurant.pos_integration === 'lightspeed' ||
                    restaurant.table_management_enabled
                  "
                  class="col-6 col-lg-4"
                >
                  <base-input label="Table management">
                    <base-switch
                      class="mr-1"
                      on-text="On"
                      off-text="Off"
                      v-model="restaurant.table_management_enabled"
                    ></base-switch>
                  </base-input>
                </div>
                <div class="col-6 col-lg-4">
                  <base-input label="Edit items from tablet menu editor">
                    <InfoIcon slot="appendLabel">
                      Enable editing items from the tablet menu editor.
                    </InfoIcon>
                    <base-switch
                      class="mr-1"
                      on-text="On"
                      off-text="Off"
                      v-model="restaurant.tablet_menu_editor_enabled"
                    ></base-switch>
                  </base-input>
                </div>
              </div>
              <div class="row">
                <div class="col-4" v-if="restaurant.payment_methods.includes('ideal')">
                  <base-input label="Pass on the iDEAL transaction cost to guests">
                    <InfoIcon slot="appendLabel">
                      This will pass on the iDEAL transaction fee to the guests. Due to
                      regulations it's not possible to pass on the costs for other payment
                      methods.
                    </InfoIcon>
                    <base-switch
                      class="mr-1"
                      on-text="On"
                      off-text="Off"
                      v-model="restaurant.guest_pay_ideal"
                    ></base-switch>
                  </base-input>
                </div>
                <div class="col-12">
                  <h6 class="heading-small text-muted mb-2">Tipping</h6>

                  <div class="row">
                    <div class="col-auto">
                      <base-input label="Tipping">
                        <base-switch
                          class="mr-1"
                          on-text="On"
                          off-text="Off"
                          v-model="restaurant.tipping_enabled"
                        >
                        </base-switch>
                      </base-input>
                    </div>

                    <div class="col-auto">
                      <base-input label="Tipping type">
                        <el-select
                          v-model="restaurant.tipping_type"
                          :disabled="!restaurant.tipping_enabled"
                          class="select-danger"
                        >
                          <el-option
                            v-for="method in [
                              {
                                value: 'Percentage',
                                label: 'Percentages (5% / 10% / 15%)',
                              },
                              { value: 'Amount', label: 'Rounded amounts' },
                            ]"
                            :key="method.value"
                            :label="method.label"
                            :value="method.value"
                          >
                          </el-option>
                        </el-select>
                      </base-input>
                    </div>
                  </div>
                </div>
              </div>
              <div v-if="isAdmin" class="row mt-2">
                <div class="col-12">
                  <BaseCodeEditor v-model="restaurant.custom_css" />
                </div>
              </div>
            </form>
          </validation-observer>
        </div>
        <div slot="footer" style="float: right">
          <base-button
            type="secondary"
            native-type="submit"
            @click="resetComponent"
            :disabled="!changesMade"
            >Cancel changes
          </base-button>
          <base-button
            type="primary"
            native-type="submit"
            @click="submitWithValidation"
            :disabled="!changesMade"
            >Save changes
          </base-button>
        </div>
      </card>
      <!--      <BaseSettingsCard title="Restaurant" saveButtonsInHeader>-->
      <!--        -->
      <!--      </BaseSettingsCard>-->

      <!--      <BaseSettingsCard title="Menu settings" saveButtonsInHeader>-->
      <!--        -->

      <!--      </BaseSettingsCard>-->

      <!--      <BaseSettingsCard title="Other settings">-->
      <!--        -->
      <!--      </BaseSettingsCard>-->

      <!--      <form @submit.prevent="">-->

      <!--      </form>-->

      <!--      <base-button type="secondary" native-type="submit" @click="resetComponent" :disabled="!changesMade">Cancel changes</base-button>-->
      <!--      <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save changes</base-button>-->
    </div>
    <div v-else class="d-flex justify-content-center py-8">
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>
  </div>
</template>

<script>
import "flatpickr/dist/flatpickr.css";
import { rawTimeZones } from "@vvo/tzdb";
import FontPicker from "font-picker-vue";
import flatPicker from "vue-flatpickr-component";
import WeektimesTable from "@/components/WeektimesTable";
import ColorPicker from "@/components/Inputs/ColorPicker";
import submitWIthValidation from "@/mixins/submitWIthValidation";
import EditVATPercentages from "../../components/EditVATPercentages";
import restaurantSettingsForm from "@/mixins/restaurantSettingsForm";
import DropzoneFileUpload from "@/components/Inputs/DropzoneFileUpload";
import EditCourses from "@/components/EditCourses";
import BaseCodeEditor from "@/components/Inputs/BaseCodeEditor";

export default {
  mixins: [restaurantSettingsForm, submitWIthValidation],
  name: "GeneralSettings",
  data: function () {
    return {
      languages: [
        {
          iso639: "af",
          iso3166: "ZA",
          name: "Afrikaans",
          native: "Afrikaans",
          emoji: "🇿🇦",
        },
        {
          iso639: "ak",
          iso3166: "GH",
          name: "Akan",
          native: "Akana",
          emoji: "🇬🇭",
        },
        {
          iso639: "sq",
          iso3166: "AL",
          name: "Albanian",
          native: "Shqip",
          emoji: "🇦🇱",
        },
        {
          iso639: "ar",
          iso3166: "AE",
          name: "Arabic",
          native: "العربية",
          emoji: "🇦🇪",
        },
        {
          iso639: "as",
          iso3166: "IN",
          name: "Assamese",
          native: "অসমীয়া",
          emoji: "🇮🇳",
        },
        {
          iso639: "az",
          iso3166: "AZ",
          name: "Azerbaijani",
          native: "Azərbaycanca / آذربايجان",
          emoji: "🇦🇿",
        },
        {
          iso639: "bm",
          iso3166: "ML",
          name: "Bambara",
          native: "Bamanankan",
          emoji: "🇲🇱",
        },
        {
          iso639: "eu",
          iso3166: "ES",
          name: "Basque",
          native: "Euskara",
          emoji: "🇪🇸",
        },
        {
          iso639: "be",
          iso3166: "BY",
          name: "Belarusian",
          native: "Беларуская",
          emoji: "🇧🇾",
        },
        {
          iso639: "bn",
          iso3166: "BD",
          name: "Bengali",
          native: "বাংলা",
          emoji: "🇧🇩",
        },
        {
          iso639: "bs",
          iso3166: "BA",
          name: "Bosnian",
          native: "Bosanski",
          emoji: "🇧🇦",
        },
        {
          iso639: "br",
          iso3166: "FR",
          name: "Breton",
          native: "Brezhoneg",
          emoji: "🇫🇷",
        },
        {
          iso639: "bg",
          iso3166: "BG",
          name: "Bulgarian",
          native: "Български",
          emoji: "🇧🇬",
        },
        {
          iso639: "my",
          iso3166: "MM",
          name: "Burmese",
          native: "မြန်မာစာ",
          emoji: "🇲🇲",
        },
        {
          iso639: "km",
          iso3166: "KH",
          name: "Cambodian",
          native: "ភាសាខ្មែរ",
          emoji: "🇰🇭",
        },
        {
          iso639: "ca",
          iso3166: "ES",
          name: "Catalan",
          native: "Català",
          emoji: "🇪🇸",
        },
        {
          iso639: "zh",
          iso3166: "CN",
          name: "Chinese",
          native: "中文",
          emoji: "🇨🇳",
        },
        {
          iso639: "kw",
          iso3166: "GB",
          name: "Cornish",
          native: "Kernewek",
          emoji: "🇬🇧",
        },
        {
          iso639: "hr",
          iso3166: "HR",
          name: "Croatian",
          native: "Hrvatski",
          emoji: "🇭🇷",
        },
        {
          iso639: "cs",
          iso3166: "CZ",
          name: "Czech",
          native: "Čeština",
          emoji: "🇨🇿",
        },
        {
          iso639: "da",
          iso3166: "DK",
          name: "Danish",
          native: "Dansk",
          emoji: "🇩🇰",
        },
        {
          iso639: "nl",
          iso3166: "NL",
          name: "Dutch",
          native: "Nederlands",
          emoji: "🇳🇱",
        },
        {
          iso639: "en",
          iso3166: "GB",
          name: "English",
          native: "English",
          emoji: "🇬🇧",
        },
        {
          iso639: "eo",
          iso3166: null,
          name: "Esperanto",
          native: "Esperanto",
          emoji: "🏳️",
        },
        {
          iso639: "et",
          iso3166: "EE",
          name: "Estonian",
          native: "Eesti",
          emoji: "🇪🇪",
        },
        {
          iso639: "ee",
          iso3166: "GH",
          name: "Ewe",
          native: "Ɛʋɛ",
          emoji: "🇬🇭",
        },
        {
          iso639: "fo",
          iso3166: "FO",
          name: "Faroese",
          native: "Føroyskt",
          emoji: "🇫🇴",
        },
        {
          iso639: "hy",
          iso3166: "AM",
          name: "Fijian",
          native: "Na Vosa Vakaviti",
          emoji: "🇦🇲",
        },
        {
          iso639: "fi",
          iso3166: "FI",
          name: "Finnish",
          native: "Suomi",
          emoji: "🇫🇮",
        },
        {
          iso639: "fr",
          iso3166: "FR",
          name: "French",
          native: "Français",
          emoji: "🇫🇷",
        },
        {
          iso639: "gl",
          iso3166: "ES",
          name: "Galician",
          native: "Galego",
          emoji: "🇪🇸",
        },
        {
          iso639: "lg",
          iso3166: "UG",
          name: "Ganda",
          native: "Luganda",
          emoji: "🇺🇬",
        },
        {
          iso639: "ka",
          iso3166: "GE",
          name: "Georgian",
          native: "ქართული",
          emoji: "🇬🇪",
        },
        {
          iso639: "de",
          iso3166: "DE",
          name: "German",
          native: "Deutsch",
          emoji: "🇩🇪",
        },
        {
          iso639: "el",
          iso3166: "GR",
          name: "Greek",
          native: "Ελληνικά",
          emoji: "🇬🇷",
        },
        {
          iso639: "kl",
          iso3166: "GL",
          name: "Greenlandic",
          native: "Kalaallisut",
          emoji: "🇬🇱",
        },
        {
          iso639: "gu",
          iso3166: "IN",
          name: "Gujarati",
          native: "ગુજરાતી",
          emoji: "🇮🇳",
        },
        {
          iso639: "ha",
          iso3166: "NG",
          name: "Hausa",
          native: "هَوُسَ",
          emoji: "🇳🇬",
        },
        {
          iso639: "he",
          iso3166: "IL",
          name: "Hebrew",
          native: "עברית",
          emoji: "🇮🇱",
        },
        {
          iso639: "hi",
          iso3166: "IN",
          name: "Hindi",
          native: "हिन्दी",
          emoji: "🇮🇳",
        },
        {
          iso639: "hu",
          iso3166: "HU",
          name: "Hungarian",
          native: "Magyar",
          emoji: "🇭🇺",
        },
        {
          iso639: "is",
          iso3166: "IS",
          name: "Icelandic",
          native: "Íslenska",
          emoji: "🇮🇸",
        },
        {
          iso639: "ig",
          iso3166: "NG",
          name: "Igbo",
          native: "Igbo",
          emoji: "🇳🇬",
        },
        {
          iso639: "id",
          iso3166: "ID",
          name: "Indonesian",
          native: "Bahasa Indonesia",
          emoji: "🇮🇩",
        },
        {
          iso639: "ga",
          iso3166: "IE",
          name: "Irish",
          native: "Gaeilge",
          emoji: "🇮🇪",
        },
        {
          iso639: "it",
          iso3166: "IT",
          name: "Italian",
          native: "Italiano",
          emoji: "🇮🇹",
        },
        {
          iso639: "ja",
          iso3166: "JP",
          name: "Japanese",
          native: "日本語",
          emoji: "🇯🇵",
        },
        {
          iso639: "kn",
          iso3166: "IN",
          name: "Kannada",
          native: "ಕನ್ನಡ",
          emoji: "🇮🇳",
        },
        {
          iso639: "kk",
          iso3166: "KZ",
          name: "Kazakh",
          native: "Қазақша",
          emoji: "🇰🇿",
        },
        {
          iso639: "ki",
          iso3166: "KE",
          name: "Kikuyu",
          native: "Gĩkũyũ",
          emoji: "🇰🇪",
        },
        {
          iso639: "rn",
          iso3166: "BI",
          name: "Kirundi",
          native: "Kirundi",
          emoji: "🇧🇮",
        },
        {
          iso639: "ko",
          iso3166: "KR",
          name: "Korean",
          native: "한국어",
          emoji: "🇰🇷",
        },
        {
          iso639: "ku",
          iso3166: null,
          name: "Kurdish",
          native: "Kurdî / كوردی",
          emoji: "🏳️",
        },
        {
          iso639: "lv",
          iso3166: "LV",
          name: "Latvian",
          native: "Latviešu",
          emoji: "🇱🇻",
        },
        {
          iso639: "ln",
          iso3166: "CG",
          name: "Lingala",
          native: "Lingála",
          emoji: "🇨🇬",
        },
        {
          iso639: "lt",
          iso3166: "LT",
          name: "Lithuanian",
          native: "Lietuvių",
          emoji: "🇱🇹",
        },
        {
          iso639: "lu",
          iso3166: "CD",
          name: "Luba-Katanga",
          native: "Tshiluba",
          emoji: "🇨🇩",
        },
        {
          iso639: "mk",
          iso3166: "MK",
          name: "Macedonian",
          native: "Македонски",
          emoji: "🇲🇰",
        },
        {
          iso639: "mg",
          iso3166: "MG",
          name: "Malagasy",
          native: "Malagasy",
          emoji: "🇲🇬",
        },
        {
          iso639: "ms",
          iso3166: "MY",
          name: "Malay",
          native: "Bahasa Melayu",
          emoji: "🇲🇾",
        },
        {
          iso639: "ml",
          iso3166: "IN",
          name: "Malayalam",
          native: "മലയാളം",
          emoji: "🇮🇳",
        },
        {
          iso639: "mt",
          iso3166: "MT",
          name: "Maltese",
          native: "bil-Malti",
          emoji: "🇲🇹",
        },
        {
          iso639: "gv",
          iso3166: "GB",
          name: "Manx",
          native: "Gaelg",
          emoji: "🇬🇧",
        },
        {
          iso639: "mr",
          iso3166: "IN",
          name: "Marathi",
          native: "मराठी",
          emoji: "🇮🇳",
        },
        {
          iso639: "mn",
          iso3166: "MN",
          name: "Mongolian",
          native: "Монгол",
          emoji: "🇲🇳",
        },
        {
          iso639: "ne",
          iso3166: "NP",
          name: "Nepali",
          native: "नेपाली",
          emoji: "🇳🇵",
        },
        {
          iso639: "nd",
          iso3166: "ZW",
          name: "North Ndebele",
          native: "Sindebele",
          emoji: "🇿🇼",
        },
        {
          iso639: "no",
          iso3166: "NO",
          name: "Norwegian",
          native: "Norsk",
          emoji: "🇳🇴",
        },
        {
          iso639: "or",
          iso3166: "IN",
          name: "Oriya",
          native: "ଓଡ଼ିଆ",
          emoji: "🇮🇳",
        },
        {
          iso639: "om",
          iso3166: "ET",
          name: "Oromo",
          native: "Oromoo",
          emoji: "🇪🇹",
        },
        {
          iso639: "pa",
          iso3166: "PK",
          name: "Panjabi / Punjabi",
          native: "ਪੰਜਾਬੀ / पंजाबी / پنجابي",
          emoji: "🇵🇰",
        },
        {
          iso639: "ps",
          iso3166: "AF",
          name: "Pashto",
          native: "پښتو",
          emoji: "🇦🇫",
        },
        {
          iso639: "fa",
          iso3166: "IR",
          name: "Persian",
          native: "فارسی",
          emoji: "🇮🇷",
        },
        {
          iso639: "ff",
          iso3166: "CN",
          name: "Peul",
          native: "Fulfulde",
          emoji: "🇨🇳",
        },
        {
          iso639: "pl",
          iso3166: "PL",
          name: "Polish",
          native: "Polski",
          emoji: "🇵🇱",
        },
        {
          iso639: "pt",
          iso3166: "PT",
          name: "Portuguese",
          native: "Português",
          emoji: "🇵🇹",
        },
        {
          iso639: "rm",
          iso3166: "CH",
          name: "Raeto Romance",
          native: "Rumantsch",
          emoji: "🇨🇭",
        },
        {
          iso639: "ro",
          iso3166: "RO",
          name: "Romanian",
          native: "Română",
          emoji: "🇷🇴",
        },
        {
          iso639: "ru",
          iso3166: "RU",
          name: "Russian",
          native: "Русский",
          emoji: "🇷🇺",
        },
        {
          iso639: "rw",
          iso3166: "RW",
          name: "Rwandi",
          native: "Kinyarwandi",
          emoji: "🇷🇼",
        },
        {
          iso639: "sg",
          iso3166: "CF",
          name: "Sango",
          native: "Sängö",
          emoji: "🇨🇫",
        },
        {
          iso639: "sr",
          iso3166: "RS",
          name: "Serbian",
          native: "Српски",
          emoji: "🇷🇸",
        },
        {
          iso639: "sn",
          iso3166: "ZW",
          name: "Shona",
          native: "chiShona",
          emoji: "🇿🇼",
        },
        {
          iso639: "ii",
          iso3166: "CN",
          name: "Sichuan Yi",
          native: "ꆇꉙ / 四川彝语",
          emoji: "🇨🇳",
        },
        {
          iso639: "si",
          iso3166: "LK",
          name: "Sinhalese",
          native: "සිංහල",
          emoji: "🇱🇰",
        },
        {
          iso639: "sk",
          iso3166: "SK",
          name: "Slovak",
          native: "Slovenčina",
          emoji: "🇸🇰",
        },
        {
          iso639: "sl",
          iso3166: "SI",
          name: "Slovenian",
          native: "Slovenščina",
          emoji: "🇸🇮",
        },
        {
          iso639: "so",
          iso3166: "SO",
          name: "Somalia",
          native: "Soomaaliga",
          emoji: "🇸🇴",
        },
        {
          iso639: "es",
          iso3166: "ES",
          name: "Spanish",
          native: "Español",
          emoji: "🇪🇸",
        },
        {
          iso639: "sw",
          iso3166: "TZ",
          name: "Swahili",
          native: "Kiswahili",
          emoji: "🇹🇿",
        },
        {
          iso639: "sv",
          iso3166: "SE",
          name: "Swedish",
          native: "Svenska",
          emoji: "🇸🇪",
        },
        {
          iso639: "ta",
          iso3166: "IN",
          name: "Tamil",
          native: "தமிழ்",
          emoji: "🇮🇳",
        },
        {
          iso639: "te",
          iso3166: "IN",
          name: "Telugu",
          native: "తెలుగు",
          emoji: "🇮🇳",
        },
        {
          iso639: "th",
          iso3166: "TH",
          name: "Thai",
          native: "ไทย / Phasa Thai",
          emoji: "🇹🇭",
        },
        {
          iso639: "bo",
          iso3166: "CN",
          name: "Tibetan",
          native: "བོད་ཡིག / Bod skad",
          emoji: "🇨🇳",
        },
        {
          iso639: "ti",
          iso3166: "ER",
          name: "Tigrinya",
          native: "ትግርኛ",
          emoji: "🇪🇷",
        },
        {
          iso639: "to",
          iso3166: "TO",
          name: "Tonga",
          native: "Lea Faka-Tonga",
          emoji: "🇹🇴",
        },
        {
          iso639: "tr",
          iso3166: "TR",
          name: "Turkish",
          native: "Türkçe",
          emoji: "🇹🇷",
        },
        {
          iso639: "uk",
          iso3166: "UA",
          name: "Ukrainian",
          native: "Українська",
          emoji: "🇺🇦",
        },
        {
          iso639: "ur",
          iso3166: "PK",
          name: "Urdu",
          native: "اردو",
          emoji: "🇵🇰",
        },
        {
          iso639: "uz",
          iso3166: "UZ",
          name: "Uzbek",
          native: "Ўзбек",
          emoji: "🇺🇿",
        },
        {
          iso639: "vi",
          iso3166: "VN",
          name: "Vietnamese",
          native: "Tiếng Việt",
          emoji: "🇻🇳",
        },
        {
          iso639: "cy",
          iso3166: "GB",
          name: "Welsh",
          native: "Cymraeg",
          emoji: "🇬🇧",
        },
        {
          iso639: "fy",
          iso3166: "NL",
          name: "West Frisian",
          native: "Frysk",
          emoji: "🇳🇱",
        },
        {
          iso639: "yo",
          iso3166: "NG",
          name: "Yoruba",
          native: "Yorùbá",
          emoji: "🇳🇬",
        },
        {
          iso639: "zu",
          iso3166: "ZA",
          name: "Zulu",
          native: "isiZulu",
          emoji: "🇿🇦",
        },
      ],
      availableServices: [
        {
          value: "waiter",
          label: "Request waiter",
          description:
            "Adds a button to the services options which sends a request to the tablet that guests would like someone to come to their table",
        },
        {
          value: "request",
          label: "Written requests",
          description:
            "Adds a button to the services options in your menu where guests can enter a short text (max. 140 chars) which they can send to the Orderli tablet as a request",
        },
        {
          value: "search",
          label: "Search ",
          description:
            "This adds a search button to your menu so guests can easily search everything on the menu (not available on takeaway/delivery menu)",
        },
        {
          value: "bill",
          label: "Request the bill",
          description:
            "Adds a button to the services options which sends a request to the tablet that the guests would like to receive the bill",
        },
        {
          value: "viewbill",
          label: "View the bill",
          description:
            "Guests can view the open bill for their table and if payment is enabled they can also pay directly. This feature is only supported with POS integration for some POS systems, contact us for more information.",
        },
      ],
      availableServicesOld: [
        "waiter",
        "bill",
        "request",
        "pin-only",
        "cash-only",
        "pay-at-pos",
        "pin-at-pos",
        "cash-at-pos",
        "viewbill",
        "search",
      ],
      imageFile: [],
      hubspotLoading: false,
      hubspotResults: [],
      hubspotDealsLoading: false,
      hubspotDealsResults: [],
      moneybirdLoading: false,
      moneybirdResults: [],
      font: null,
      fontFamilyMain: null,
      fontFamilyHeading: null,
      configureItemTypes: false,
      createItemTypeInput: null,
      creatingItemType: false,
      customDomainStatus: null,
    };
  },
  components: {
    EditCourses,
    DropzoneFileUpload,
    WeektimesTable,
    flatPicker,
    FontPicker,
    ColorPicker,
    EditVATPercentages,
    BaseCodeEditor,
  },
  mounted() {
    //Kan dit niet in de mounted hook doen, want deze weet niet of this.restaurant al wel bestond
    // if (!this.restaurant.main_color) this.restaurant.main_color = "#d64f28"
    // if (!this.restaurant.accent_color) this.restaurant.accent_color = "#c5deb8"
    if (this.currentRestaurant.custom_hostname !== null) {
      this.getCustomDomainStatus();
    }
  },
  computed: {
    currentAccountId() {
      return this.$store.getters.currentAccountId;
    },
    currentAccount() {
      return this.$store.getters.currentAccount;
    },
    currentRestaurant() {
      return this.$store.getters.currentRestaurant;
    },
    currentRestaurantId() {
      return this.$store.getters.currentRestaurantId;
    },
    isAdmin() {
      return this.$store.getters.isAdmin;
    },
    jwt() {
      return localStorage.getItem("jwt");
    },
    apiURL() {
      return this.$store.state.apiURL;
    },
    timezones() {
      let timezones = rawTimeZones;
      timezones.push({ name: "CET", raw: "CET" });
      return timezones;
    },
    itemTypes() {
      return this.$store.state.menu.itemTypes;
    },
  },
  watch: {
    "restaurant.facebook_pixel": function (newVal) {
      if (newVal === "") {
        this.restaurant.facebook_pixel = null;
      }
    },
    "restaurant.database_name": function (newVal) {
      this.restaurant.database_name = newVal.replace(/\W-/g, "");
    },
  },
  methods: {
    uploadLogo() {
      alert("changing logo still work in progress");
    },
    checkFile(file) {
      if (
        file.type !== "image/jpeg" &&
        file.type !== "image/png" &&
        file.type !== "image/gif"
      ) {
        this.$refs.imageDropzone.dropzone.removeFile(file);
        alert(
          `Filetype '${file.type}' not supported. Please upload images in .jpeg, .png, or .gif format. Contact us if you need additional formats.`
        );
      } else if (file.size > 200000) {
        this.$refs.imageDropzone.dropzone.removeFile(file);
        alert(
          `Image too big. Image is ${Math.round(
            file.size / 1000
          )}kb, should be max. 200kb`
        );
      }
    },
    placeholderImgUrl() {
      console.log("failed to load logo");
    },
    nameKeydown(e) {
      if (/^\W-$/.test(e.key)) {
        e.preventDefault();
      }
    },
    searchHubspot(query) {
      let self = this;
      if (query !== "") {
        this.hubspotLoading = true;
        this.$store
          .dispatch("searchHubspot", query)
          .then(function (res) {
            self.hubspotLoading = false;
            self.hubspotResults = res.data;
          })
          .catch(function (err) {
            self.hubspotLoading = false;
            alert("something went wrong");
            console.log(err);
          });
      } else {
        this.hubspotResults = [];
      }
    },
    searchHubspotDeals(query) {
      let self = this;
      if (query !== "") {
        this.hubspotDealsLoading = true;
        this.$store
          .dispatch("searchHubspotDeals", query)
          .then(function (res) {
            self.hubspotDealsLoading = false;
            self.hubspotDealsResults = res.data;
          })
          .catch(function (err) {
            self.hubspotDealsLoading = false;
            alert("something went wrong");
            console.log(err);
          });
      } else {
        this.hubspotDealsResults = [];
      }
    },
    searchMoneybird(query) {
      let self = this;
      if (query !== "") {
        this.moneybirdLoading = true;
        this.$store
          .dispatch("searchMoneybird", query)
          .then(function (res) {
            self.moneybirdLoading = false;
            self.moneybirdResults = res.data;
          })
          .catch(function (err) {
            self.moneybirdLoading = false;
            alert("An error occurred");
            console.log(err);
          });
      } else {
        this.moneybirdResults = [];
      }
    },
    fontChosenHeading(font) {
      this.font = font;
      this.fontFamilyHeading = font.family;
    },
    fontChosenMain(font) {
      this.font = font;
      this.fontFamilyMain = font.family;
    },
    updateItemType(type) {
      this.$store
        .dispatch("updateItemType", type)
        .then(function (res) {
          alert(`updated name to ${type.name}!`);
        })
        .catch(function (err) {
          self.creatingItemType = false;
          alert("something went wrong");
          console.log(err);
        });
    },
    createItemType() {
      let self = this;
      self.creatingItemType = true;
      this.$store
        .dispatch("createItemType", { name: this.createItemTypeInput })
        .then(function (res) {
          self.creatingItemType = false;
          self.createItemTypeInput = "";
        })
        .catch(function (err) {
          self.creatingItemType = false;
          alert("something went wrong");
          console.log(err);
        });
    },
    authenticateMailchimp() {
      this.$store
        .dispatch("authenticateMailchimp")
        .then(function (res) {})
        .catch(function (err) {
          alert("Something went wrong");
        });
    },
    authenticateUntappd() {
      this.$store
        .dispatch("authenticateUntappd")
        .then(function (res) {})
        .catch(function (err) {
          alert("Something went wrong");
        });
    },
    createStripeSubscription() {
      let self = this;
      this.$store
        .dispatch("createStripeSubscription")
        .then(function (res) {
          self.restaurant.stripe_subscription_id = "";
          alert("Success");
        })
        .catch(function (err) {
          alert(err);
        });
    },
    getCustomDomainStatus() {
      let self = this;
      this.$store
        .dispatch("getCustomDomainStatus")
        .then((res) => {
          self.customDomainStatus = res.data.status;
        })
        .catch((err) => {});
    },
    formatToTimestring(secondsfrommidnight) {
      if (isNaN(secondsfrommidnight)) {
        return null;
      }
      if (secondsfrommidnight === undefined) return null;
      if (secondsfrommidnight >= 86400) {
        secondsfrommidnight = 0;
      }
      let h = Math.floor(secondsfrommidnight / 3600);
      let m = Math.floor((secondsfrommidnight % 3600) / 60);
      //TODO IMPORTANT this function is triggered on every change in the form / card item
      return ("0" + h).slice(-2) + ":" + ("0" + m).slice(-2);
    },
    formatToSecondsTimeslot(e) {
      let timeInput = e.target.value.split(":");
      let seconds = +timeInput[0] * 60 * 60 + +timeInput[1] * 60;
      this.restaurant.custom_midnight_time = seconds;
    },
  },
};
</script>

<style scoped>
.fontPicker /deep/ > button {
  background: #fff !important;
}

.fontPicker /deep/ ul.expanded {
  background: #fff;
  max-height: 400px;
}

.customPadding::v-deep .input-group-text {
  padding: 0rem 0.75rem;
}
</style>
