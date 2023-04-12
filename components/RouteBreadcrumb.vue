<template>
  <bread-crumb list-classes="breadcrumb-links">
    <BreadCrumbItem class="restaurant-name">
      {{ currentRestaurantName }}
    </BreadCrumbItem>

    <BreadCrumbItem class="active">
      {{ currentRouteName }}
    </BreadCrumbItem>
  </bread-crumb>
</template>

<script>
  import BreadCrumb from './Breadcrumb';
  import BreadCrumbItem from './BreadcrumbItem';

  export default {
    name: 'route-breadcrumb',
    components: {
      BreadCrumb,
      BreadCrumbItem
    },
    computed: {
      currentRestaurantName() {
        return this.$store.getters.currentRestaurant.name;
      },
      currentRouteName() {
        const lastRoute = this.$route.matched.slice().pop();

        return lastRoute && lastRoute.name;
      }
    },
    methods: {
      getBreadName(route) {
        return route.name;
      }
    }
  };
</script>

<style scoped>
  .restaurant-name{
    font-weight: 600;
    line-height: 1.5;
    color: #d64f28;
  }
</style>
