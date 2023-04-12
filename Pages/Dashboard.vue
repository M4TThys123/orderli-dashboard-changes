<template>
  <div>
    <base-header class="pb-6" type="">
      <!--      <div class="row align-items-center py-4">-->
      <!--        <div class="col-lg-6 col-7">-->
      <!--          <h6 class="h2 d-inline-block mb-0">Default</h6>-->
      <!--          <nav aria-label="breadcrumb" class="d-none d-md-inline-block ml-md-4">-->
      <!--            <route-bread-crumb></route-bread-crumb>-->
      <!--          </nav>-->
      <!--        </div>-->
      <!--        <div class="col-lg-6 col-5 text-right">-->
      <!--          <base-button size="sm" type="neutral">New</base-button>-->
      <!--          <base-button size="sm" type="neutral">Filters</base-button>-->
      <!--        </div>-->
      <!--      </div>-->

      <!-- Card stats -->
      <!--      <div class="row pt-4">-->
      <!--        <div class="col-xl-3 col-md-6">-->
      <!--          <stats-card title="Total traffic"-->
      <!--                      type="gradient-red"-->
      <!--                      sub-title="XXX,XXX"-->
      <!--                      icon="ni ni-active-40">-->

      <!--            <template slot="footer">-->
      <!--              <span class="text-success mr-2"><i class="fa fa-arrow-up"></i> X.XX%</span>-->
      <!--              <span class="text-nowrap">Since last month</span>-->
      <!--            </template>-->
      <!--          </stats-card>-->
      <!--        </div>-->
      <!--        <div class="col-xl-3 col-md-6">-->
      <!--          <stats-card title="Total traffic"-->
      <!--                      type="gradient-orange"-->
      <!--                      sub-title="X,XXX"-->
      <!--                      icon="ni ni-chart-pie-35">-->

      <!--            <template slot="footer">-->
      <!--              <span class="text-success mr-2"><i class="fa fa-arrow-up"></i> XX.XX%</span>-->
      <!--              <span class="text-nowrap">Since last month</span>-->
      <!--            </template>-->
      <!--          </stats-card>-->
      <!--        </div>-->
      <!--        <div class="col-xl-3 col-md-6">-->
      <!--          <stats-card title="Sales"-->
      <!--                      type="gradient-green"-->
      <!--                      sub-title="XXX"-->
      <!--                      icon="ni ni-money-coins">-->

      <!--            <template slot="footer">-->
      <!--              <span class="text-danger mr-2"><i class="fa fa-arrow-down"></i> X.XX%</span>-->
      <!--              <span class="text-nowrap">Since last month</span>-->
      <!--            </template>-->
      <!--          </stats-card>-->

      <!--        </div>-->
      <!--        <div class="col-xl-3 col-md-6">-->
      <!--          <stats-card title="Performance"-->
      <!--                      type="gradient-info"-->
      <!--                      sub-title="XX,XX%"-->
      <!--                      icon="ni ni-chart-bar-32">-->

      <!--            <template slot="footer">-->
      <!--              <span class="text-success mr-2"><i class="fa fa-arrow-up"></i> XX.X%</span>-->
      <!--              <span class="text-nowrap">Since last month</span>-->
      <!--            </template>-->
      <!--          </stats-card>-->
      <!--        </div>-->
      <!--      </div>-->
    </base-header>
    <div class="container-fluid mt--1">
      <div class="row starter-page col-md-12 pr-0">
        <!--        <p class="font-weight-normal">Welcome to the Orderli dashboard. The dashboard is still in beta, thanks for your early interest! Please leave any feedback through the <router-link to="/support">Support</router-link> page, or send an email to <a href="mailto:support@orderli.com">support@orderli.com</a>.</p>-->
        <h2>{{ goodDayMessage }}, {{ userFirstName }} 👋🏼</h2>
        <br />
        <p class="font-weight-normal">
          Welcome to the Orderli dashboard. The dashboard is still in active development,
          thanks for your early interest! Please send any feedback to your contact person,
          or send an email to
          <a href="mailto:support@orderli.com">support@orderli.com</a>.
        </p>
      </div>

      <div class="row">
        <div class="col-md-6">
          <card v-if="mandateGiven === false && (canManageAccounts || role === 'owner')">
            <h2 class="m-0" slot="header">⚠️ Mandate not completed</h2>
            <div>
              A mandate for a SEPA Direct Debit is required to use Orderli. Please
              configure the mandate in billing settings. We use SEPA Direct Debit to
              collect the monthly subscription fee of Orderli and the transaction costs of
              any online payments done through Orderli.
            </div>
            <base-button
              slot="footer"
              @click="$router.push('/settings#billing')"
              icon
              type="primary"
              class="float-right"
              >Go to settings</base-button
            >
          </card>
        </div>
      </div>

      <only admins>
        <div class="row mt-4">
          <div class="col-6">
            <card header-classes="bg-transparent">
              <div slot="header" class="row align-items-center w-100">
                <div class="col">
                  <h6 class="text-uppercase text-muted ls-1 mb-1">Recommendations</h6>
                  <h5 class="h3 mb-0">Cross sell products</h5>
                </div>
                <div class="col-12 pt-3">
                  <unlock-feature feature="cross_selling">
                    <RecommendationsTable :showCompact="true" />
                  </unlock-feature>
                </div>
              </div>
            </card>
          </div>
        </div>
      </only>

    </div>
  </div>
</template>
<script>
  import RecommendationsTable from "@/views/Products/RecommendationsTable";

  export default {
    name: 'starter-page',
    components: {
      RecommendationsTable
    },
    computed: {
      userFirstName() {
        return this.$store.getters.userFirstName;
      },
      goodDayMessage() {
        let hour = new Date().getHours();
        return (
          "Good " +
          ((hour < 12 && hour >= 5 && "morning") ||
            (hour < 18 && hour >= 12 && "afternoon") ||
            (hour < 24 && hour >= 18 && "evening") ||
            "night")
        );
      },
    },
    canManageAccounts() {
      return this.$store.getters.canManageAccounts;
    },
    mandateGiven() {
      return this.$store.getters.mandateGiven;
    },
    role() {
      return this.$store.getters.currentRole;
    },
  }
</script>
<style>
.starter-page {
  /*min-height: calc(100vh - 380px);*/
}
</style>
