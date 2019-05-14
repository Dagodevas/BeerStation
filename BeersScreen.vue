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
        @click.stop="$router.push({ name: 'app/beers/add'})"
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
            @click.stop="deleteBeers"
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
        <v-list
          v-if="beers.length > 0"
          two-line
        >
          <v-layout
            row
            wrap
            v-for="beer in beers"
            :key="beer.id"
          >
            <v-flex
              d-flex
              v-bind:style="{ backgroundColor: (beer.deleteValue && del_switch) ?  'lightslategray'  : '' , opacity: (beer.deleteValue && del_switch) ? '0.8' : '1'}"
            >
              <beer-item
                :clicable="!del_switch"
                :beer="beer"
                :subtitle="getLocalizedDescription($root, beer.description).value"
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
import { beersQuery } from "@/graphql/queries";
import { APP_ROUTER } from "@/models/constants";
import Beer from "@/models/Beer";
import { getLocalizedDescription } from "@/helpers/localization";
import BeerActions from "@/vuex/actions/BeerActions";
import BreweryActions from "@/vuex/actions/BreweryActions";
import BeerItem from "@/components/BeerItem.vue";

@Component({
  components: {
    BeerItem
  },

  methods: {
    getLocalizedDescription
  }
})
export default class BeersScreen extends Vue {
  get beers(): Beer[] {
    const normalizedString = (this.searchValue || "").toLowerCase();
    return this.$store.getters.getFilteredBeers(
      (beer: Beer) =>
        beer!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
        beer!.description!.some(
          description =>
            description.value!.toLowerCase().indexOf(normalizedString) > -1
        ) ||
        beer!.brewery!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
        beer!.type!.toLowerCase().indexOf(normalizedString) > -1
    );
  }

  private searchValue: string = "";
  private del_switch: boolean = false;

  public created() {
    this.$store.dispatch(BeerActions.getBeers.name);
    this.$store.dispatch(BreweryActions.getBreweries.name);
  }

  private toggleDelete() {
    this.del_switch = !this.del_switch;
    if (!this.del_switch) {
      this.beers.forEach(beer => (beer.deleteValue = false));
    }
  }

  private deleteBeers() {
    const beers = this.beers.filter(beer => beer.deleteValue === true);

    this.$store
      .dispatch(BeerActions.deleteBeers.name, beers)
      .then(() => this.$store.dispatch(BeerActions.getBeers.name));
  }
}
</script>
