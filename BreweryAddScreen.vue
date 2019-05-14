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
        <v-toolbar-title>{{ $t("add_new_brewery") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!breweryModified"
        small
        color="success"
        @click.stop="saveBrewery"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <BreweryForm
      v-if="readOnlyBrewery"
      ref="breweryForm"
      :brewery="readOnlyBrewery"
      :onBreweryChange="onBreweryChange"
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
import { Brewery } from "@/models";
import BreweryForm from "@/components/BreweryForm.vue";
import BreweryActions from "@/vuex/actions/BreweryActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    BreweryForm
  }
})
export default class BreweryAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyBrewery: Brewery = new AttributeInitializer(
    new Brewery()
  ).initLocalizedAttribute("description").item;
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

  private saveBrewery() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(BreweryActions.createBrewery.name, this.brewery)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const brewery = data.profile.admin.createBrewery;
      this.$store.dispatch(BreweryActions.setBrewery.name, brewery).then(() => {
        this.$router.replace(APP_ROUTER.BREWERIES.PATH);
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.breweryModified = false;
  }
}
</script>
