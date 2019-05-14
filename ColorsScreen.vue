<template>
  <v-container>
    <v-toolbar app>
      <v-toolbar-side-icon @click.stop="$store.dispatch('toggleDrawer')"></v-toolbar-side-icon>
      <v-flex
        mt-2
        xs12
      >
        <v-text-field
          solo
          clearable
          flat
          label="Search"
          v-model="searchValue"
          prepend-inner-icon="search"
        ></v-text-field>
      </v-flex>
      <v-btn
        icon
        @click.stop="$router.push({ name: 'app/colors/add'})"
      >
        <v-icon>{{ "add" }}</v-icon>
      </v-btn>
      <v-btn
        icon
        @click.stop="toggleDelete"
      >
        <v-icon>{{ "fa-trash-alt" }}</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
    </v-toolbar>

    <v-layout column>
      <!-- <div v-if="$apollo.loading">
        <p>Loading</p>
      </div> -->
      <div v-if="del_switch">
        <v-layout
          row
          align-center
        >
          <div class="text-xs-center">
            <v-btn
              round
              color="green"
              @click.stop="deleteColors"
              dark
            >{{$t("confirm")}}</v-btn>
          </div>

          <div class="text-xs-center">
            <v-btn
              round
              color="red"
              @click.stop="toggleDelete"
              dark
            >{{$t("cancel")}}</v-btn>
          </div>
        </v-layout>
      </div>

      <v-list
        v-if="colors.length > 0"
        two-line
      >
        <v-layout
          row
          wrap
          v-for="color in colors"
          :key="color.id"
        >
          <v-flex
            d-flex
            v-bind:style="{ backgroundColor: (color.deleteValue && del_switch) ?  'lightslategray'  : '' , opacity: (color.deleteValue && del_switch) ? '0.8' : '1'}"
          >
            <color-item
              :clicable="!del_switch"
              :color="color"
              :subtitle="color.value"
            />
          </v-flex>
        </v-layout>
      </v-list>
      <div v-else>
        Not found
      </div>
    </v-layout>
  </v-container>
</template>

          



<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { APP_ROUTER } from "@/models/constants";
import ColorPicker from "vue-color-picker-wheel";
import ColorItem from "@/components/ColorItem.vue";
import ColorActions from "@/vuex/actions/ColorActions";
import { colorsQuery } from "@/graphql/queries";
import Color from "@/models/Color";
import { getLocalizedDescription } from "@/helpers/localization";

@Component({
  components: {
    ColorPicker,
    ColorItem
  },

  methods: {
    getLocalizedDescription
  }
})
export default class ColorsScreen extends Vue {
  get colors(): Color[] {
    const normalizedString = (this.searchValue || "").toLowerCase();
    return this.$store.getters.getFilteredColors(
      (color: Color) =>
        color!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
        color!.value!.toLowerCase().indexOf(normalizedString) > -1
    );
  }

  private searchValue: string = "";
  private del_switch: boolean = false;

  public created() {
    this.$store.dispatch(ColorActions.getColors.name);
  }

  private toggleDelete() {
    this.del_switch = !this.del_switch;
    if (!this.del_switch) {
      this.colors.forEach(color => (color.deleteValue = false));
    }
  }
}
</script>
