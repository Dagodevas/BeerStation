<template>
  <v-container v-if="readOnlyBeer">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyBeer && readOnlyBeer.name ? readOnlyBeer.name : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!beerModified"
        small
        color="success"
        @click.stop="saveBeer(false)"
      >SAVE</v-btn>

      <!-- <v-tooltip v-if="beerModified == true" bottom>
        <v-icon slot="activator" color="yellow">warning</v-icon>
        <span>There are unsaved changes</span>
      </v-tooltip> -->

    </v-toolbar>
    <BeerForm
      ref="beerForm"
      :beer="readOnlyBeer"
      :onBeerChange="onBeerChange"
    />

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveBeer(true)"
    />
  </v-container>
  <div
    v-else
    class="text-xs-center"
  >
    <v-progress-circular
      color="primary"
      indeterminate
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Provide } from "vue-property-decorator";
import { APP_ROUTER } from "@/models/constants";
import { Beer } from "@/models";
import BeerForm from "@/components/BeerForm.vue";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import BeerActions from "../vuex/actions/BeerActions";
import { Validator } from "vee-validate";

@Component({
  components: {
    BeerForm,
    MergeConflictModal
  }
})
export default class BeerDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyBeer(): Beer {
    return new AttributeInitializer(
      this.$store.getters.getBeerById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.BEERS.PATH;

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

  private saveBeer(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? BeerActions.updateBeer.name
              : BeerActions.updateBeerConcurrent.name,
            this.beer
          )
          .then(this.processUpdateResponse)
          .catch(error => {
            if (error.graphQLErrors.some(x => x.code === "modified")) {
              this.$refs.modal.open();
            }
          });
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const beer =
        data.profile.admin.updateBeerConcurrent ||
        data.profile.admin.updateBeer;
      this.$store.dispatch(BeerActions.setBeer.name, beer).then(() => {
        this.$refs.modal.close();
        this.$forceUpdate();
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.beerModified = false;
  }
}
</script>
