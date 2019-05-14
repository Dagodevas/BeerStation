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
        @click.stop="$router.push({ name: 'app/complements/add'})"
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

    <div v-if="del_switch">
      <v-layout
        row
        align-center
      >
        <div class="text-xs-center">
          <v-btn
            round
            color="green"
            @click.stop="deleteComplements"
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

    <v-layout column>
      <!-- <div v-if="$apollo.loading">
        <p>Loading</p>
      </div> -->
      <div>
        <v-list
          v-if="complements.length > 0"
          two-line
        >
          <v-layout
            row
            wrap
            v-for="complement in complements"
            :key="complement.id"
          >

            <v-flex
              d-flex
              v-bind:style="{ backgroundColor: (complement.deleteValue && del_switch) ?  'lightslategray'  : '' , opacity: (complement.deleteValue && del_switch) ? '0.8' : '1'}"
            >
              <complement-item
                :clicable="!del_switch"
                :complement="complement"
                :subtitle="complement.description ? getLocalizedDescription($root, complement.description).value : '' "
              />
            </v-flex>

          </v-layout>
        </v-list>
        <div v-else>
          Not found
        </div>
      </div>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { complementsQuery } from "@/graphql/queries";
import { APP_ROUTER } from "@/models/constants";
import Complement from "@/models/Complement";
import { getLocalizedDescription } from "@/helpers/localization";
import BeerActions from '@/vuex/actions/BeerActions';
import ComplementActions from '@/vuex/actions/ComplementActions';
import ComplementItem from "@/components/ComplementItem.vue";

@Component({

 components: {
    ComplementItem
  },

  methods: {
    getLocalizedDescription
  }
})
export default class ComplementsScreen extends Vue {
    get complements(): Complement[] {
        const normalizedString = (this.searchValue || "").toLowerCase();
        return this.$store.getters.getFilteredComplements(
            (complement: Complement) => (
                1
                // complement!.name!.toLowerCase().indexOf(normalizedString) > -1 ||
                // complement!.description!.some(description => description.value!.toLowerCase().indexOf(normalizedString) > -1)
            )
        );
    }

    private searchValue: string = "";
    private del_switch: boolean = false;
      
    public created() {
        this.$store.dispatch(ComplementActions.getComplements.name);
    }

    private toggleDelete(){
      this.del_switch = !this.del_switch
      if(!this.del_switch){
        this.complements.forEach((complement)=>
          complement.deleteValue=false
        )
      }
    }

    private deleteComplements() {
    
      const complements= this.complements.filter((complement)=>complement.deleteValue===true);

      this.$store.dispatch(
          ComplementActions.deleteComplements.name,complements
      ).then(()=>this.$store.dispatch(ComplementActions.getComplements.name))
   }
}
</script>
