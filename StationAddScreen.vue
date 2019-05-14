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
        <v-toolbar-title>{{ $t("add_new_station") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!stationModified"
        small
        station="success"
        @click.stop="saveStation"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <StationForm
      v-if="readOnlyStation"
      ref="stationForm"
      :station="readOnlyStation"
      :onStationChange="onStationChange"
    />
    <div
      v-else
      class="text-xs-center"
    >
      <v-progress-circular
        station="primary"
        indeterminate
      />
    </div>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue, Provide } from "vue-property-decorator";
import { APP_ROUTER } from "@/models/constants";
import { Station } from "@/models";
import StationForm from "@/components/StationForm.vue";
import StationActions from "@/vuex/actions/StationActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    StationForm
  }
})
export default class StationAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyStation: Station = new AttributeInitializer(
    new Station()
  ).initLocalizedAttribute("description").item;
  private station: Station = new Station();
  private stationModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.stationModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onStationChange(changed: boolean, station: Station) {
    this.stationModified = changed;
    this.station = station;
  }

  private saveStation() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(StationActions.createStation.name, this.station)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const station = data.profile.admin.createStation;
      this.$store.dispatch(StationActions.setStation.name, station).then(() => {
        this.$router.replace(APP_ROUTER.STATIONS.PATH);
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.stationModified = false;
  }
}
</script>
