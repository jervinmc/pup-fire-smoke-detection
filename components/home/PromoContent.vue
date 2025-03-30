<template>
  <v-sheet class="py-10">
    <v-slide-y-transition>
    <div class="text-h5 pl-10 v-leave-active" v-show="isShow">
       Latest Items
    </div>
    </v-slide-y-transition>
    <v-slide-group v-model="model" class="pa-4" multiple show-arrows>
      <v-slide-item v-for="n in 15" :key="n" >
        <v-card :loading="loading" class="mx-5 my-5" max-width="300">
          <template slot="progress">
            <v-progress-linear
              color="deep-purple"
              height="10"
              indeterminate
            ></v-progress-linear>
          </template>
          <v-img
            height="150"
            src="https://cdn.vuetifyjs.com/images/cards/cooking.png"
          ></v-img>
          <v-card-title>Cafe Badilico</v-card-title>
          <v-card-text>
            <v-row align="center" class="mx-0">
              <v-rating
                :value="4.5"
                color="amber"
                dense
                half-increments
                readonly
                size="14"
              ></v-rating>
              <div class="grey--text ms-4">4.5 (413)</div>
            </v-row>

            <div class="my-4 text-subtitle-1"><b>10 points</b></div>

            <div>
              Small plates, salads & sandwiches - an intimate setting with 12
              indoor seats plus patio seating.
            </div>
          </v-card-text>
          <v-divider class="mx-4"></v-divider>
          <v-card-actions>
            <v-btn color="secondary--text" text @click="$Location('redeem')">
             <b class="secondary--text"> Get </b>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-slide-item>
    </v-slide-group>
  </v-sheet>
</template>

<script>
export default {
    created(){
        this.isShow = true
    },
    data(){
        return{
            isShow:false,
        }
    }
};
</script>

<style>
</style>