<template>
  <div>
    <hero />
    <div>
      <promo-content />
    </div>
    <div>
      <about />
    </div>
    <div>

    </div>
  </div>
</template>

<script>
import About from '../components/home/About.vue'
import Hero from '../components/home/Hero.vue'
import PromoContent from '../components/home/PromoContent.vue'
export default {
  components: { Hero, About, PromoContent  },

}
</script>

<style>

</style>