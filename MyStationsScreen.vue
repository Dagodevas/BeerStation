<template>
  <v-container>
    <v-toolbar app>
      <v-toolbar-side-icon @click.stop="$store.dispatch('toggleDrawer')"></v-toolbar-side-icon>
      <v-flex
        mt-2
        xs12
      >
        <v-text-field
          solo
          clearable
          flat
          label="Search"
          v-model="searchValue"
          prepend-inner-icon="search"
        ></v-text-field>
      </v-flex>
      <v-btn
        icon
        @click.stop="$router.push({ name: 'app/stations/add'})"
      >
        <v-icon>{{ "add" }}</v-icon>
      </v-btn>
      <v-btn
        icon
        @click.stop="toggleDelete"
      >
        <v-icon>{{ "fa-trash-alt" }}</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
    </v-toolbar>
    <v-layout
      row
      wrap
    >
      <v-flex
        d-flex
        xs12
        sm6
        md4
        v-for="station in stations"
        :key="station.id"
      >
        <StationOverview :station="station" />
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
import { Station, Account } from "@/models";
import { Component, Vue } from "vue-property-decorator";
import StationOverview from "@/components/dashboard/StationOverview.vue";

@Component({
  components: {
    StationOverview
  }
})
export default class MyStationsScreen extends Vue {
  get stations(): Station[] {
    const normalizedString = (this.searchValue || "").toLowerCase();
    return this.$store.getters.getFilteredmyStations(
      (station: Station) =>
        station!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
        station!.address!.toLowerCase().indexOf(normalizedString) > -1 ||
        station!.country!.toLowerCase().indexOf(normalizedString) > -1
    );
  }

  private searchValue: string = "";
  private del_switch: boolean = false;

  private toggleDelete() {
    this.del_switch = !this.del_switch;
  }
}
</script>