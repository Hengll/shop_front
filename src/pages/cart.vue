<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <h1 class="text-center">{{ $t('nav.cart') }}</h1>
      </v-col>
      <v-divider></v-divider>
      <v-col cols="12">
        <v-list lines="two">
          <template v-for="(item, i) in cart" :key="item._id">
            <v-list-item
              :title="item.product.name"
              :subtitle="item.product.price"
              active-class="bg-red"
              :active="!item.product.sell"
            >
              <template #prepend>
                <v-avatar rounded="0">
                  <v-img :src="item.product.image"></v-img>
                </v-avatar>
              </template>
              <template #append>
                <v-btn
                  icon="mdi-minus"
                  class="me-2 btnn"
                  @click="editCart(item.product._id, -1)"
                ></v-btn>
                {{ item.quantity }}
                <v-btn
                  icon="mdi-plus"
                  class="ms-2 btnn"
                  @click="editCart(item.product._id, 1)"
                ></v-btn>
              </template>
            </v-list-item>
            <v-divider v-if="i < cart.length - 1"></v-divider>
          </template>
        </v-list>
      </v-col>
      <v-col cols="12" class="text-center">
        <p>{{ total }}</p>
        <v-btn color="primary" :disabled="!canCheckout" @click="checkout">{{
          $t('cart.checkout')
        }}</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useAxios } from '@/composables/axios'
import { useUserStore } from '@/stores/user'
import { useRouter } from 'vue-router'
import { useSnackbar } from 'vuetify-use-dialog'
import { useI18n } from 'vue-i18n'

const { apiAuth } = useAxios()
const user = useUserStore()
const router = useRouter()
const createSnackbar = useSnackbar()
const { t } = useI18n()

const cart = ref([])

const getCart = async () => {
  try {
    const { data } = await apiAuth.get('/user/cart')
    cart.value = data.result
  } catch (err) {
    console.log(err)
  }
}
getCart()

const total = computed(() => {
  return cart.value.reduce((acc, item) => acc + item.product.price * item.quantity, 0)
})

const canCheckout = computed(() => {
  return cart.value.length > 0 && cart.value.every((item) => item.product.sell)
})

const editCart = async (product, quantity) => {
  try {
    const { data } = await apiAuth.patch('/user/cart', { product, quantity })
    user.cart = data.result
    getCart()
  } catch (error) {
    console.log(error)
  }
}

const checkout = async () => {
  try {
    await apiAuth.post('/order')
    user.cart = 0
    router.push('/orders')
  } catch (error) {
    console.log(error)
    createSnackbar({
      text: t('api.' + (error?.response?.data?.message || 'unknownError')),
      snackbarProps: {
        color: 'red',
      },
    })
  }
}
</script>

<route lang="json">
{
  "meta": {
    "login": true,
    "admin": false,
    "title": "nav.cart"
  }
}
</route>

<style lang="scss" scoped>
.btnn {
  border-radius: 5px;
  height: 30px;
  border: 1px solid #999999;
  color: lightblue;

  & + & {
    color: lightcoral;
  }
}
</style>
