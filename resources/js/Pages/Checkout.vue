<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head, router, useForm } from '@inertiajs/vue3';
import { computed, onMounted, toRefs, ref } from 'vue';

import { useCartStore } from '@/store/cart'
import { storeToRefs } from 'pinia'
const cartStore = useCartStore()
const { cart } = storeToRefs(cartStore)

const props = defineProps({ intent: Object, order: Object })
const { intent } = toRefs(props)

let stripe = null
let elements = null
let card = null
let form = null
let isProcessing = ref(false)
const data = useForm({ payment_intent: null })

onMounted(() => {
  stripe = Stripe("pk_test_51Q4x8MB4L3iPOj7LJb64Dhe3FKEVYYVnwTOmG822xB35GsWhomTecPHGZwjXtf3ld9mg5n9FnYK6xqQBaRLtzLnc009NQoYAW7");

  elements = stripe.elements();
    let style = {
      base: {
        color: "#32325d",
        fontFamily: 'Arial, sans-serif',
        fontSmoothing: "antialiased",
        fontSize: "16px",
        "::placeholder": {
          color: "#32325d"
        }
      },
      invalid: {
        fontFamily: 'Arial, sans-serif',
        color: "#fa755a",
        iconColor: "#fa755a"
      }
    };

    card = elements.create("card", { style: style });
    // Stripe injects an iframe into the DOM
    card.mount("#card-element");

    card.on("loaderror", function (event) {
      // Listen for load errors, handling any errors using your own observability tooling.
    });

    card.on("change", function (event) {
      // Disable the Pay button if there are no card details in the Element
      document.querySelector("button").disabled = event.empty;
      document.querySelector("#card-error").textContent = event.error ? event.error.message : "";
    });

    var form = document.getElementById("payment-form");
    form.addEventListener("submit", function(event) {
      event.preventDefault();
      // Complete payment when the submit button is clicked
      payWithCard(stripe, card, intent.value.client_secret);
    });

    setTimeout(() => {
      router.post('/checkout', {
        total: totalWithoutDot(),
        total_decimal: total,
        items: cart.value
      })
    }, 10)
})

// Calls stripe.confirmCardPayment
// If the card requires authentication Stripe shows a pop-up modal to
// prompt the user to enter authentication details without leaving your page.
const payWithCard = (stripe, card, clientSecret) => {
  loading(true);
  stripe
    .confirmCardPayment(clientSecret, {
      payment_method: {
        card: card
      }
    })
    .then(function(result) {
      if (result.error) {
        // Show error to your customer
        showError(result.error.message);
      } else {
        // The payment succeeded!
        orderComplete(result.paymentIntent.id);
      }
    });
};

/* ------- UI helpers ------- */

// Shows a success message when the payment is complete
const orderComplete = (paymentIntentId) => {
  data.payment_intent = paymentIntentId
  data.put('/checkout')
};

// Show the customer the error from Stripe if their card fails to charge
let showError = (errorMsgText) => {
  loading(false);
  var errorMsg = document.querySelector("#card-error");
  errorMsg.textContent = errorMsgText;
  setTimeout(function() {
    errorMsg.textContent = "";
  }, 4000);
};

// Show a spinner on payment submission
let loading = (isLoading) => {
  if (isLoading) {
    // Disable the button and show a spinner
    document.querySelector("button").disabled = true;
    isProcessing.value = true
  } else {
    document.querySelector("button").disabled = false;
    isProcessing.value = false
  }
};

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
      <div class="p-4 mt-2 max-w-[1250px] mx-auto text-3xl font-extrabold">Checkout</div>

      <div class="flex max-w-[1250px] mx-auto pt-4">
        <div class="w-8/12">
          <div class="flex items-stretch border-b border-b-gray-300 w-[calc(100%-100px)] pb-4 pl-4 mb-6">
            <div class="text-gray-900 font-extrabold text-xl mr-12">
              Shipping Address
            </div>
            <div class="px-4 font-semibold">
              <div>{{ $page.props.auth.user.first_name }} {{ $page.props.auth.user.last_name }}</div>
              <div>{{ $page.props.auth.address.addr1 }}</div>
              <div>{{ $page.props.auth.address.addr2 }}</div>
              <div>{{ $page.props.auth.address.city }}</div>
              <div>{{ $page.props.auth.address.postcode }}</div>
              <div>{{ $page.props.auth.address.country }}</div>
            </div>
          </div>
        </div>

        <div class="w-4/12 border border-gray-400 rounded-md py-4 px-2">
          <form id="payment-form">
            <div id="card-element">
              <!-- Stripe.js injects the Card Element -->
            </div>

            <div class="flex justify-between text-xl text-red-700 font-extrabold border-y border-y-gray-300 my-3 p-2">
              <div>Order total:</div>
              <div v-if="order">USD: {{ order.total_decimal }}</div>
            </div>
            <button id="submit" class="bg-yellow-400 hover:bg-yellow-500 rounded-md text-sm font-extrabold p-2">
              <div v-if="isProcessing" id="button-text">Processing...</div>
              <div v-else id="button-text">Place your order in USD</div>
            </button>

            <p id="card-error" role="alert" class="text-red-700 text-center font-semibold"></p>
          </form>
        </div>
      </div>

      <div class="w-[1200px] mx-auto text-xl font-bold pb-2 underline">Items</div>
      <div class="w-[1200px] mx-auto" v-for="prod in JSON.parse(order.items)" :key="prod">
        <div class="flex items-center py-1">
          <img width="60" :src="prod.image" class="rounded-md" alt="">
          <div class="ml-4">
            <div class="text-lg font-semibold">{{ prod.title }}</div>
            <div class="font-semibold text-red-700">${{ prod.price }}</div>
          </div>
        </div>
      </div>
    </AuthenticatedLayout>
</template>

<style>
#card-element {
  border-radius: 4px 4px 0 0 ;
  padding: 12px;
  border: 1px solid rgba(50, 50, 93, 0.1);
  height: 44px;
  width: 100%;
  background: white;
}

#payment-request-button {
  margin-bottom: 32px;
}
</style>