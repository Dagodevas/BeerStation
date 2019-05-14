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
        @click.stop="$router.push({ name: 'app/breweries/add'})"
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

    <div v-if="del_switch">
      <v-layout
        row
        align-center
      >
        <div class="text-xs-center">
          <v-btn
            round
            color="green"
            @click.stop="deleteBreweries"
            dark
          >{{$t("confirm")}}</v-btn>
        </div>

        <div class="text-xs-center">
          <v-btn
            round
            color="red"
            @click.stop="toggleDelete"
            dark
          >{{$t("cancel")}}</v-btn>
        </div>
      </v-layout>
    </div>

    <v-layout column>
      <!-- <div v-if="$apollo.loading">
        <p>Loading</p>
      </div> -->
      <div>
        {{$store.breweries}}
        <v-list
          v-if="breweries.length > 0"
          two-line
        >
          <v-layout
            row
            wrap
            v-for="brewery in breweries"
            :key="brewery.id"
          >

            <v-flex
              d-flex
              v-bind:style="{ backgroundColor: (brewery.deleteValue && del_switch) ?  'lightslategray'  : '' , opacity: (brewery.deleteValue && del_switch) ? '0.8' : '1'}"
            >
              <brewery-item
                :clicable="!del_switch"
                :brewery="brewery"
                :subtitle="brewery.country"
              />
            </v-flex>

          </v-layout>
        </v-list>
        <div v-else>
          Not found
        </div>
      </div>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { breweriesQuery } from "@/graphql/queries";
import Brewery from "@/models/Brewery";
import { APP_ROUTER } from "@/models/constants";
import { getLocalizedDescription } from "@/helpers/localization";
import BreweryActions from "@/vuex/actions/BreweryActions";
import BreweryItem from "@/components/BreweryItem.vue";

@Component({
  components: {
    BreweryItem
  },

  methods: {
    getLocalizedDescription
  }
})
export default class BreweriesScreen extends Vue {
  get breweries(): Brewery[] {
    const normalizedString = (this.searchValue || "").toLowerCase();
    return this.$store.getters.getFilteredBreweries(
      (brewery: Brewery) =>
        brewery!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
        brewery!.country!.toLowerCase().indexOf(normalizedString) > -1
    );
  }

  private searchValue: string = "";
  private del_switch: boolean = false;

  public created() {
    this.$store.dispatch(BreweryActions.getBreweries.name);
  }

  private toggleDelete() {
    this.del_switch = !this.del_switch;
    if (!this.del_switch) {
      this.breweries.forEach(brewery => (brewery.deleteValue = false));
    }
  }

  private deleteBreweries() {
    const breweries = this.breweries.filter(
      brewery => brewery.deleteValue === true
    );

    this.$store
      .dispatch(BreweryActions.deleteBrewery.name, breweries)
      .then(() => this.$store.dispatch(BreweryActions.getBreweries.name));
  }
}
</script>






