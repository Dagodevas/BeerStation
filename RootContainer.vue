<template>
  <v-content>
    <div v-if="$store.state.ready">
      <v-navigation-drawer :clipped="clipped" v-model="$store.state.drawerVisibility" enable-resize-watcher fixed app>
        <DashboardMenu />
      </v-navigation-drawer>

      <router-view drawer />
    </div>
    <div v-else class="text-xs-center">
      <v-progress-circular color="primary" indeterminate />
    </div>
  </v-content>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator"
import DashboardMenu from "@/components/DashboardMenu.vue"

import { images } from "@/themes"

@Component({
  components: {
    DashboardMenu,
  },
})
export default class RootContainer extends Vue {
    private clipped: boolean = false;
    private fixed: boolean = false;
    private miniVariant: boolean = false;
}
</script>