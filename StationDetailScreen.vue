
<template>
  <v-container v-if="readOnlyStation">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyStation && readOnlyStation.name ? readOnlyStation.name : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!stationModified"
        small
        station="success"
        @click.stop="saveStation(false)"
      >SAVE</v-btn>

      <!-- <v-tooltip v-if="stationModified == true" bottom>
        <v-icon slot="activator" station="yellow">warning</v-icon>
        <span>There are unsaved changes</span>
      </v-tooltip> -->

    </v-toolbar>
    <StationForm
      ref="stationForm"
      :station="readOnlyStation"
      :onStationChange="onStationChange"
    />

    <!-- <div
      row
      wrap
      v-for="board in readOnlyStation.boards"
      :key="board.id"
    >

      {{this.$store.getters.myBoardBeers(board)}}

    </div> -->

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveStation(true)"
    />
  </v-container>
  <div
    v-else
    class="text-xs-center"
  >
    <v-progress-circular
      station="primary"
      indeterminate
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Provide } from "vue-property-decorator";
import { APP_ROUTER } from "@/models/constants";
import { Station } from "@/models";
import StationForm from "@/components/StationForm.vue";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import StationActions from "../vuex/actions/StationActions";
import { getLocalizedDescription } from "@/helpers/localization";
import { Validator } from "vee-validate";

@Component({
  methods: {
    getLocalizedDescription
  },
  components: {
    StationForm,
    MergeConflictModal
  }
})
export default class StationDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyStation(): Station {
    return new AttributeInitializer(
      this.$store.getters.getmyStationById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.STATIONS.PATH;

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

  private saveStation(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? StationActions.updateStation.name
              : StationActions.updateStationConcurrent.name,
            this.station
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
      const station =
        data.profile.admin.updateStationConcurrent ||
        data.profile.admin.updateStation;
      this.$store.dispatch(StationActions.setStation.name, station).then(() => {
        this.$refs.modal.close();
        this.$forceUpdate();
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.stationModified = false;
  }
}
</script>
