<template>
  <v-container v-if="readOnlyBeerType">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyBeerType && readOnlyBeerType.name ? readOnlyBeerType.name : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!beerTypeModified"
        small
        color="success"
        @click.stop="saveBeerType(false)"
      >SAVE</v-btn>

      <!-- <v-tooltip v-if="beerTypeModified == true" bottom>
        <v-icon slot="activator" color="yellow">warning</v-icon>
        <span>There are unsaved changes</span>
      </v-tooltip> -->

    </v-toolbar>
    <BeerTypeForm
      ref="beerTypeForm"
      :beerType="readOnlyBeerType"
      :onBeerTypeChange="onBeerTypeChange"
    />

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveBeerType(true)"
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
import { BeerType } from "@/models";
import BeerTypeForm from "@/components/BeerTypeForm.vue";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import BeerTypeActions from "../vuex/actions/BeerTypeActions";
import { Validator } from "vee-validate";

@Component({
  components: {
    BeerTypeForm,
    MergeConflictModal
  }
})
export default class BeerTypeDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyBeerType(): BeerType {
    return new AttributeInitializer(
      this.$store.getters.getBeerTypeById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.BEERTYPES.PATH;

  private beerType: BeerType = new BeerType();
  private beerTypeModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.beerTypeModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onBeerTypeChange(changed: boolean, beerType: BeerType) {
    this.beerTypeModified = changed;
    this.beerType = beerType;
  }

  private saveBeerType(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? BeerTypeActions.updateBeerType.name
              : BeerTypeActions.updateBeerTypeConcurrent.name,
            this.beerType
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
      const beerType =
        data.profile.admin.updateBeerTypeConcurrent ||
        data.profile.admin.updateBeerType;
      this.$store
        .dispatch(BeerTypeActions.setBeerType.name, beerType)
        .then(() => {
          this.$refs.modal.close();
          this.$forceUpdate();
        });
    } catch (e) {
      alert("Error processing server response");
    }

    this.beerTypeModified = false;
  }
}
</script>
