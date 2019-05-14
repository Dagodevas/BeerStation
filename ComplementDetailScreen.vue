<template>
  <v-container v-if="readOnlyComplement">
    <v-toolbar app>
      <v-btn
        icon
        @click.stop="$router.back()"
      >
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-flex xs12>
        <v-toolbar-title>{{readOnlyComplement && readOnlyComplement.name ? getLocalizedDescription($root, readOnlyComplement.name).value : ""}}</v-toolbar-title>
      </v-flex>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!complementModified"
        small
        color="success"
        @click.stop="saveComplement(false)"
      >SAVE</v-btn>

      <!-- <v-tooltip v-if="complementModified == true" bottom>
        <v-icon slot="activator" color="yellow">warning</v-icon>
        <span>There are unsaved changes</span>
      </v-tooltip> -->

    </v-toolbar>
    <ComplementForm
      ref="complementForm"
      :complement="readOnlyComplement"
      :onComplementChange="onComplementChange"
    />

    <MergeConflictModal
      ref="modal"
      :onDiscard="() => $router.replace(returnPath)"
      :onForcePush="() => saveComplement(true)"
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
import { Complement } from "@/models";
import ComplementForm from "@/components/ComplementForm.vue";
import AttributeInitializer from "@/helpers/AttributeInitializer";
import MergeConflictModal from "@/components/MergeConflictModal.vue";
import ComplementActions from "../vuex/actions/ComplementActions";
import { getLocalizedDescription } from "@/helpers/localization";
import { Validator } from "vee-validate";

@Component({
  methods: {
    getLocalizedDescription
  },
  components: {
    ComplementForm,
    MergeConflictModal
  }
})
export default class ComplementDetailScreen extends Vue {
  @Provide("validator")
  $validator: Validator = this.$validator;

  get readOnlyComplement(): Complement {
    return new AttributeInitializer(
      this.$store.getters.getComplementById(this.$router.currentRoute.params.id)
    ).initLocalizedAttribute("description").item;
  }

  private returnPath = APP_ROUTER.COMPLEMENTS.PATH;

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

  private saveComplement(force: boolean) {
    this.$validator.validate().then((valid: Boolean) => {
      if (valid) {
        this.$store
          .dispatch(
            force
              ? ComplementActions.updateComplement.name
              : ComplementActions.updateComplementConcurrent.name,
            this.complement
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
      const complement =
        data.profile.admin.updateComplementConcurrent ||
        data.profile.admin.updateComplement;
      this.$store
        .dispatch(ComplementActions.setComplement.name, complement)
        .then(() => {
          this.$refs.modal.close();
          this.$forceUpdate();
        });
    } catch (e) {
      alert("Error processing server response");
    }

    this.complementModified = false;
  }
}
</script>
