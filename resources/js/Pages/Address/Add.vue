<template>
  <Head title="Address" />

  <AuthenticatedLayout>
    <div class="mt-8"></div>

    <div class="max-w-[500px] mx-auto text-2xl font-extrabold">
      <div>Add a new address</div>

      <form>
        <div class="text-[15px] -mb-1.5 font-extrabold">Country</div>
        <select 
          class="
            w-full
            border border-gray-300
            rounded-lg
            shadow-md
            py-1
            bg-gray-200
            hover:bg-gray-300
            cursor-pointer
            focus:border-orange-400
            focus:ring-orange-400
          "
          name="country"
        >
          <option selected value="United Kingdom">United Kingdom</option>
          <option value="United States">United States</option>
          <option value="Germany">Germany</option>
          <option value="France">France</option>
        </select>
      </form>
    </div>
  </AuthenticatedLayout>
</template>

<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head, Link } from '@inertiajs/vue3';
import { toRefs } from 'vue';
import MapMarkerOutlineIcon from 'vue-material-design-icons/MapMarkerOutline.vue';
import PlusIcon from 'vue-material-design-icons/Plus.vue';

</script>
