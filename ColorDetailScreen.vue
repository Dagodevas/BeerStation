<template>
  <v-container v-if="readOnlyColor">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyColor && readOnlyColor.name ? readOnlyColor.name : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!colorModified"
        small
        color="success"
        @click.stop="saveColor(false)"
      >SAVE</v-btn>

      <!-- <v-tooltip v-if="colorModified == true" bottom>
        <v-icon slot="activator" color="yellow">warning</v-icon>
        <span>There are unsaved changes</span>
      </v-tooltip> -->

    </v-toolbar>
    <ColorForm
      ref="colorForm"
      :color="readOnlyColor"
      :onColorChange="onColorChange"
    />

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveColor(true)"
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
import { Color } from "@/models";
import ColorForm from "@/components/ColorForm.vue";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import ColorActions from "../vuex/actions/ColorActions";
import { getLocalizedDescription } from "@/helpers/localization";
import { Validator } from "vee-validate";

@Component({
  methods: {
    getLocalizedDescription
  },
  components: {
    ColorForm,
    MergeConflictModal
  }
})
export default class ColorDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyColor(): Color {
    return new AttributeInitializer(
      this.$store.getters.getColorById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.COLORS.PATH;

  private color: Color = new Color();
  private colorModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.colorModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onColorChange(changed: boolean, color: Color) {
    this.colorModified = changed;
    this.color = color;
  }

  private saveColor(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? ColorActions.updateColor.name
              : ColorActions.updateColorConcurrent.name,
            this.color
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
      const color =
        data.profile.admin.updateColorConcurrent ||
        data.profile.admin.updateColor;
      this.$store.dispatch(ColorActions.setColor.name, color).then(() => {
        this.$refs.modal.close();
        this.$forceUpdate();
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.colorModified = false;
  }
}
</script>
