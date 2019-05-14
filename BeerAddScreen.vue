<template>
  <v-container>
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{ $t("add_new_beer") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!beerModified"
        small
        color="success"
        @click.stop="saveBeer"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <BeerForm
      v-if="readOnlyBeer"
      ref="beerForm"
      :beer="readOnlyBeer"
      :onBeerChange="onBeerChange"
    />
    <div
      v-else
      class="text-xs-center"
    >
      <v-progress-circular
        color="primary"
        indeterminate
      />
    </div>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue, Provide } from "vue-property-decorator";
import { APP_ROUTER } from "@/models/constants";
import { Beer } from "@/models";
import BeerForm from "@/components/BeerForm.vue";
import BeerActions from "@/vuex/actions/BeerActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    BeerForm
  }
})
export default class BeerAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyBeer: Beer = new AttributeInitializer(
    new Beer()
  ).initLocalizedAttribute("description").item;
  private beer: Beer = new Beer();
  private beerModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.beerModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onBeerChange(changed: boolean, beer: Beer) {
    this.beerModified = changed;
    this.beer = beer;
  }

  private saveBeer() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(BeerActions.createBeer.name, this.beer)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const beer = data.profile.admin.createBeer;
      this.$store.dispatch(BeerActions.setBeer.name, beer).then(() => {
        this.$router.replace(APP_ROUTER.BEERS.PATH);
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.beerModified = false;
  }
}
</script>
