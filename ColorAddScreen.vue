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
        <v-toolbar-title>{{ $t("add_new_color") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!colorModified"
        small
        color="success"
        @click.stop="saveColor"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <ColorForm
      v-if="readOnlyColor"
      ref="colorForm"
      :color="readOnlyColor"
      :onColorChange="onColorChange"
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
import { Color } from "@/models";
import ColorForm from "@/components/ColorForm.vue";
import ColorActions from "@/vuex/actions/ColorActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    ColorForm
  }
})
export default class ColorAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyColor: Color = new AttributeInitializer(
    new Color()
  ).initLocalizedAttribute("description").item;
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

  private saveColor() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(ColorActions.createColor.name, this.color)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const color = data.profile.admin.createColor;
      this.$store.dispatch(ColorActions.setColor.name, color).then(() => {
        this.$router.replace(APP_ROUTER.COLORS.PATH);
      });
    } catch (e) {
      alert("Error processing server response");
    }

    this.colorModified = false;
  }
}
</script>
