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
        @click.stop="$router.push({ name: 'app/beerTypes/add'})"
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
            @click.stop="deleteBeerTypes"
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
          v-if="beerTypes.length > 0"
          two-line
        >
          <v-layout
            row
            wrap
            v-for="beerType in beerTypes"
            :key="beerType.id"
          >
            <v-flex
              d-flex
              v-bind:style="{ backgroundColor: (beerType.deleteValue && del_switch) ?  'lightslategray'  : '' , opacity: (beerType.deleteValue && del_switch) ? '0.8' : '1'}"
            >
              <beerType-item
                :clicable="!del_switch"
                :beerType="beerType"
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
import { beerTypesQuery } from "@/graphql/queries";
import { APP_ROUTER } from "@/models/constants";
import BeerType from "@/models/BeerType";
import { getLocalizedDescription } from "@/helpers/localization";
import BeerTypeActions from "@/vuex/actions/BeerTypeActions";
import BeerTypeItem from "@/components/BeerTypeItem.vue";

@Component({
  components: {
    BeerTypeItem
  },

  methods: {
    getLocalizedDescription
  }
})
export default class BeersScreen extends Vue {
  get beerTypes(): BeerType[] {
    const normalizedString = (this.searchValue || "").toLowerCase();
    return this.$store.getters.getFilteredBeerTypes(
      (beerType: BeerType) =>
        beerType!.name!.toLowerCase().indexOf(normalizedString) > -1
    );
  }

  private searchValue: string = "";
  private del_switch: boolean = false;

  public created() {
    this.$store.dispatch(BeerTypeActions.getBeerTypes.name);
  }

  private toggleDelete() {
    this.del_switch = !this.del_switch;
    if (!this.del_switch) {
      this.beerTypes.forEach(beerType => (beerType.deleteValue = false));
    }
  }

  private deleteBeers() {
    const beerTypes = this.beerTypes.filter(
      beerType => beerType.deleteValue === true
    );

    this.$store
      .dispatch(BeerTypeActions.deleteBeerTypes.name, beerTypes)
      .then(() => this.$store.dispatch(BeerTypeActions.getBeerTypes.name));
  }
}
</script>
