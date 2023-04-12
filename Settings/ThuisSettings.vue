<template>
  <div class="card p-4">
    <div v-if="currentRestaurant && restaurant">
      <div class="position-sticky d-flex top-3" style="z-index: 900">
        <base-button size="sm" class="ml-auto" type="secondary" native-type="submit" @click="resetComponent" :disabled="!changesMade">Cancel changes</base-button>
        <base-button size="sm" type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save changes</base-button>
      </div>

      <form @submit.prevent="">

        <h6 class="heading-small text-muted mb-2">Thuis settings</h6>
        <div class="row">
          <div
            v-if="currentRestaurant.pos_integration"
            class="col-auto">
            <base-input
              type="text"
              label="POS table nr"
              v-model="restaurant.thuis_table_nr"
            >
            </base-input>
          </div>
          <div class="col-auto">
            <base-input
              type="number"
              label="Timeslot interval (minutes)"
              v-model.number="restaurant.timeslot"
            >
            </base-input>
          </div>
          <div class="col-auto">
            <base-input
              type="number"
              label="Exact delivery timeslots"
            >
              <base-switch class="mr-1" on-text="Yes" off-text="No" v-model="restaurant.exact_timeslot"></base-switch>
            </base-input>
          </div>
          <div class="col-auto">
            <base-input
              type="number"
              label="Autoswipe to POS timing (minutes)"
              v-model.number="restaurant.thuis_autoswipe_timing"
            >
              <InfoIcon slot="appendLabel">This value will delay autoswiping the order to the POS untill {{restaurant.thuis_autoswipe_timing ? restaurant.thuis_autoswipe_timing : 'X'}} minutes before the requested time of the order. Leaving it empty will push new takeaway/delivery orders immediately to the POS</InfoIcon>
            </base-input>
          </div>
        </div>

        <div class="row">

          <div class="col-6">
            <h6 class="heading-small text-muted mb-2">Delivery</h6>
            <base-input
              type="text"
              label="Enable delivery"
            >
              <base-switch class="mr-1" on-text="On" off-text="Off" v-model="restaurant.delivery_enabled"></base-switch>
            </base-input>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="text"
                  label="Delivery POS table nr"
                  :disabled="!restaurant.delivery_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.delivery_table_nr"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Delivery timeslot interval"
                  min="0"
                  :disabled="!restaurant.delivery_enabled"
                  v-model.number="restaurant.delivery_timeslot"
                >
                </base-input>
              </div>
            </div>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Delivery POS table nr range from"
                  :disabled="!restaurant.delivery_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.delivery_table_nr_range_from"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Delivery POS table nr range to"
                  :disabled="!restaurant.delivery_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.delivery_table_nr_range_to"
                >
                </base-input>
              </div>
            </div>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Delivery timeslot limit"
                  min="0"
                  :disabled="!restaurant.delivery_enabled"
                  v-model.number="restaurant.timeslot_delivery_limit"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Minimum delivery time"
                  :disabled="!restaurant.delivery_enabled"
                  v-model.number="restaurant.min_delivery_time"
                >
                </base-input>
              </div>
            </div>

            <base-input
              type="text"
              label="Delivery message"
            >
              <textarea rows="2" class="form-control" :disabled="!restaurant.delivery_enabled" v-model.lazy="restaurant.delivery_message"></textarea>
            </base-input>

            <base-input
              type="text"
              label="Delivery confirmation email message"
            >
              <textarea rows="2" class="form-control" :disabled="!restaurant.delivery_enabled" v-model.lazy="restaurant.delivery_confirmation_message"></textarea>
            </base-input>

            <base-input
              label="Delivery customer info fields"
            >
              <el-select
                :disabled="!restaurant.delivery_enabled"
                v-model="restaurant.delivery_fields"
                multiple
                class="select-danger"
              >
                <el-option
                  v-for="field in [{value: 'full_name', label: 'Full name'},{value: 'phone_number', label: 'Phone number'},{value: 'email_address', label: 'Email address'}]"
                  :key="field.value"
                  :label="field.label"
                  :value="field.value">
                </el-option>
              </el-select>
            </base-input>

            <div class="row">
              <div class="col-6">
                <base-input
                  type="number"
                  label="Delivery costs)"
                  :disabled="!restaurant.delivery_enabled"
                  v-model.number="restaurant.delivery_costs"
                >
                </base-input>
              </div>
              <div class="col-6">
                <base-input
                  type="number"
                  label="Minimum delivery amount"
                  :disabled="!restaurant.delivery_enabled"
                  v-model.number="restaurant.min_delivery_amount"
                >
                </base-input>
              </div>

              <div class="col-auto">
                <base-input
                  type="text"
                  :disabled="!restaurant.delivery_enabled"
                  label="Free delivery"
                >
                  <base-switch class="mr-1" on-text="On" off-text="Off" v-model="restaurant.free_delivery_enabled"></base-switch>
                </base-input>
              </div>
              <div class="col">
                <base-input
                  type="number"
                  label="Min. free delivery amount"
                  :disabled="!(restaurant.delivery_enabled && restaurant.free_delivery_enabled)"
                  v-model.number="restaurant.min_free_delivery_amount"
                >
                </base-input>
              </div>
            </div>


            <base-input
              label="Only deliver to specific postal codes"
            >
              <tags-input
                v-model="restaurant.delivery_postal_codes"
                :disabled="!restaurant.delivery_enabled"
                placeholder='Add new postcode (e.g. "8911")'
                tagType="primary"
              ></tags-input>
            </base-input>



          </div>


          <div class="col-6">
            <h6 class="heading-small text-muted mb-2">Takeaway</h6>
            <base-input
              type="text"
              label="Enable takeaway"
            >
              <base-switch class="mr-1" on-text="On" off-text="Off" v-model="restaurant.takeaway_enabled"></base-switch>
            </base-input>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="text"
                  label="Takeaway POS table nr"
                  :disabled="!restaurant.takeaway_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.takeaway_table_nr"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Takeaway timeslot interval"
                  min="0"
                  :disabled="!restaurant.takeaway_enabled"
                  v-model.number="restaurant.takeaway_timeslot"
                >
                </base-input>
              </div>
            </div>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Takeaway POS table nr range from"
                  :disabled="!restaurant.takeaway_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.takeaway_table_nr_range_from"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Takeaway POS table nr range to"
                  :disabled="!restaurant.takeaway_enabled || currentRestaurant.pos_integration === null"
                  v-model="restaurant.takeaway_table_nr_range_to"
                >
                </base-input>
              </div>
            </div>

            <div class="row">
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Timeslot limit (takeaway)"
                  min="0"
                  :disabled="!restaurant.takeaway_enabled"
                  v-model.number="restaurant.timeslot_takeaway_limit"
                >
                </base-input>
              </div>
              <div class="col-lg-6">
                <base-input
                  type="number"
                  label="Minimum takeaway time"
                  :disabled="!restaurant.takeaway_enabled"
                  v-model.number="restaurant.min_takeaway_time"
                >
                </base-input>
              </div>
            </div>


            <base-input
              type="text"
              label="Takeaway message"
            >
              <textarea rows="1" class="form-control" :disabled="!restaurant.takeaway_enabled" v-model.lazy="restaurant.takeaway_message"></textarea>
            </base-input>
            <base-input
              type="text"
              label="Takeaway confirmation email message"
            >
              <textarea rows="2" class="form-control" :disabled="!restaurant.takeaway_enabled" v-model.lazy="restaurant.takeaway_confirmation_message"></textarea>
            </base-input>

            <base-input
              label="Takeaway customer info fields"
            >
              <el-select
                :disabled="!restaurant.takeaway_enabled"
                v-model="restaurant.takeaway_fields"
                multiple
                class="select-danger"
              >
                <el-option
                  v-for="field in [{value: 'full_name', label: 'Full name'},{value: 'phone_number', label: 'Phone number'},{value: 'email_address', label: 'Email address'}]"
                  :key="field.value"
                  :label="field.label"
                  :value="field.value">
                </el-option>
              </el-select>
            </base-input>


          </div>

        </div>


        <hr class="my-4">

      </form>

      <base-button type="secondary" native-type="submit" @click="resetComponent" :disabled="!changesMade">Cancel changes</base-button>
      <base-button type="primary" native-type="submit" @click="onSubmit" :disabled="!changesMade">Save changes</base-button>


    </div>

    <div v-else class="d-flex justify-content-center py-8">
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>


  </div>


</template>

<script>
import restaurantSettingsForm from '@/mixins/restaurantSettingsForm';
import TagsInput from '@/components/Inputs/TagsInput'

export default {
  mixins: [restaurantSettingsForm],
  components: {TagsInput},
  name: "ThuisSettings",

}
</script>

<style scoped>

</style>
