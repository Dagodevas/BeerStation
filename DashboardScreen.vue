<template>
  <!-- <v-container fluid grid-list-md> -->
 
  <v-layout row wrap>
    <v-flex d-flex xs12 sm6 md4>
      <BeersOverview />
    </v-flex>
    <v-flex d-flex xs12 sm6 md4>
      <BreweriesOverview />
    </v-flex>
    <v-flex d-flex xs12 sm6 md4>
      <ComplementsOverview />
    </v-flex>
    <v-flex d-flex xs12 sm6 md4>
      <BeersOverview />
    </v-flex>
    
  </v-layout>
  <!-- </v-container> -->
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator"
import BeersOverview from "@/components/dashboard/BeersOverview.vue";
import BreweriesOverview from "@/components/dashboard/BreweriesOverview.vue";
import ComplementsOverview from "@/components/dashboard/ComplementsOverview.vue";

@Component({
  components: {
    BeersOverview,
    BreweriesOverview,
    ComplementsOverview,
  },
})
export default class DashboardScreen extends Vue {
}
</script>