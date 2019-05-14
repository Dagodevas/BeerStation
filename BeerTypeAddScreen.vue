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
        <v-toolbar-title>{{ $t("add_new_beerType") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!beerTypeModified"
        small
        color="success"
        @click.stop="saveBeerType"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <BeerTypeForm
      v-if="readOnlyBeerType"
      ref="beerTypeForm"
      :beerType="readOnlyBeerType"
      :onBeerTypeChange="onBeerTypeChange"
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
import { BeerType } from "@/models";
import BeerTypeForm from "@/components/BeerTypeForm.vue";
import BeerTypeActions from "@/vuex/actions/BeerTypeActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    BeerTypeForm
  }
})
export default class BeerTypeAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyBeerType: BeerType = new AttributeInitializer(
    new BeerType()
  ).initLocalizedAttribute("description").item;
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

  private saveBeerType() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(BeerTypeActions.createBeerType.name, this.beerType)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const beerType = data.profile.admin.createBeerType;
      this.$store
        .dispatch(BeerTypeActions.setBeerType.name, beerType)
        .then(() => {
          this.$router.replace(APP_ROUTER.BEERTYPES.PATH);
        });
    } catch (e) {
      alert("Error processing server response");
    }

    this.beerTypeModified = false;
  }
}
</script>
