<template>
  <v-app>
    <v-main>
      <CartaoParaCotacoes
        v-for="(item, index) in lista"
        :key="index"
        :ref="'node' + index"
        :currency="item.currency"
      />
    </v-main>
  </v-app>
</template>

<script>
  import CartaoParaCotacoes from '@/components/CartaoParaCotacoes.vue'
export default {
 components: {
    CartaoParaCotacoes
  },
  data () {
    return {
      lista: [
        { currency: 'USD' },
        { currency: 'BTC' },
        { currency: 'EUR' }
      ]
    }
  }
}
</script>
