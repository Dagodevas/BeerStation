<template>
  <v-container v-if="readOnlyBrewery">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyBrewery && readOnlyBrewery.name ? readOnlyBrewery.name : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>
      <v-btn
        :disabled="!breweryModified"
        small
        color="success"
        @click.stop="saveBrewery(false)"
      >SAVE</v-btn>
    </v-toolbar>
    <BreweryForm
      ref="breweryForm"
      :brewery="readOnlyBrewery"
      :onBreweryChange="onBreweryChange"
    />

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveBrewery(true)"
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
import { Component, Provide, Vue } from "vue-property-decorator";
import { Validator } from "vee-validate";
import { breweryQuery } from "@/graphql/queries";
import { Beer, Brewery } from "@/models";
import BreweryForm from "@/components/BreweryForm.vue";
import { APP_ROUTER } from "@/models/constants";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import BreweryActions from "../vuex/actions/BreweryActions";

@Component({
  components: {
    BreweryForm,
    MergeConflictModal
  }
})
export default class BreweryDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyBrewery(): Brewery {
    return new AttributeInitializer(
      this.$store.getters.getBreweryById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.BREWERIES.PATH;

  private brewery: Brewery = new Brewery();
  private breweryModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.breweryModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onBreweryChange(changed: boolean, brewery: Brewery) {
    this.breweryModified = changed;
    this.brewery = brewery;
  }

  private saveBrewery(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? BreweryActions.updateBrewery.name
              : BreweryActions.updateBreweryConcurrent.name,
            this.brewery
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
      const brewery =
        data.profile.admin.updateBreweryConcurrent ||
        data.profile.admin.updateBrewery;
      this.$store.dispatch(BreweryActions.setBrewery.name, brewery).then(() => {
        this.$refs.modal.close();
        this.$forceUpdate();
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.breweryModified = false;
  }
}
</script>
