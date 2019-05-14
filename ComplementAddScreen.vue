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
        <v-toolbar-title>{{ $t("add_new_complement") }}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!complementModified"
        small
        color="success"
        @click.stop="saveComplement"
      >{{ $t("add") }}</v-btn>

    </v-toolbar>
    <ComplementForm
      v-if="readOnlyComplement"
      ref="complementForm"
      :complement="readOnlyComplement"
      :onComplementChange="onComplementChange"
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
import { Complement } from "@/models";
import ComplementForm from "@/components/ComplementForm.vue";
import ComplementActions from "@/vuex/actions/ComplementActions";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import { Validator } from "vee-validate";

@Component({
  components: {
    ComplementForm
  }
})
export default class ComplementAddScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  private readOnlyComplement: Complement = new AttributeInitializer(
    new Complement()
  ).initLocalizedAttribute("description").item;
  private complement: Complement = new Complement();
  private complementModified: boolean = false;

  private created() {
    window.addEventListener("beforeunload", this.checkUnsavedForm);
  }
  private destroyed() {
    window.removeEventListener("beforeunload", this.checkUnsavedForm);
  }

  private checkUnsavedForm(e: any) {
    if (!this.complementModified) {
      return;
    }
    // Cancel the event as stated by the standard.
    e.preventDefault();
    // Chrome requires returnValue to be set.
    e.returnValue = "";
  }

  private onComplementChange(changed: boolean, complement: Complement) {
    this.complementModified = changed;
    this.complement = complement;
  }

  private saveComplement() {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(ComplementActions.createComplement.name, this.complement)
          .then(this.processUpdateResponse);
      }
    });
  }

  private processUpdateResponse({ data }: { data: any }) {
    try {
      const complement = data.profile.admin.createComplement;
      this.$store
        .dispatch(ComplementActions.setComplement.name, complement)
        .then(() => {
          this.$router.replace(APP_ROUTER.COMPLEMENTS.PATH);
        });
    } catch (e) {
      alert("Error processing server response");
    }

    this.complementModified = false;
  }
}
</script>
