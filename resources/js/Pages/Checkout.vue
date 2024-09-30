<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head, Link } from '@inertiajs/vue3';
import { computed, toRefs } from 'vue';

import { useCartStore } from '@/store/cart'
import { storeToRefs } from 'pinia'
const cartStore = useCartStore()
const { cart } = storeToRefs(cartStore)

const removeFromCart = (id) => {
  cartStore.removeProductFromCart(id)
}

const total = computed(() => {
  let total = 0
  cart.value.forEach(c => {
    total += c.price
  })
  if (total > 0) {
    return total.toFixed(2)
  }
  return 0
})

const totalWithoutDot = () => {
  let num = String(total.value).split('.').join('')
  return Number(num)
}
</script>

<template>
    <Head title="Checkout" />

    <AuthenticatedLayout>
      CHECKOUUT page
    </AuthenticatedLayout>
</template>

