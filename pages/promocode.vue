<template>
  <v-container fill-height>
    <v-row class="fill-height align-center justify-center">
      <v-col v-if="!loading" class="text-center">
        <div :class="['qr-code__container', { 'qr-code--used': status === 'used' }]">
          <qr-code v-if="code" :text="code" :class="['mx-auto', 'qr-code__img']" :style="{ width: '100%', maxWidth: '256px' }"></qr-code>
          <div class="qr-code__text red--text">ใช้งานแล้ว</div>
        </div>
        <div class="text-h5 mt-3">{{ code }}</div>
      </v-col>
      <v-col v-else class="text-center">
        <v-progress-circular width="3" size="32" indeterminate />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'PromoCodePage',
  data() {
    return {
      loading: true,
      code: null,
      status: 'used'
    }
  },
  mounted() {
    // eslint-disable-next-line no-undef
    window.vConsole = new VConsole()
    if (this.$route.query.code) {
      this.validatePromoCode(this.$route.query.code)
    } else {
      this.$nuxt.error({ statusCode: 404 })
    }
  },
  methods: {
    async validatePromoCode(code) {
      try {
        this.loading = true
        const db = this.$fire.database
        const key = 'facebook/orders/' + code
        const ref = db.ref(key)
        const snapshot = await ref.once('value')
        // console.log(key, snapshot.val())
        const val = snapshot.val()
        // console.log(val)
        if (val) {
          this.code = val.order_code
          this.status = val.status
        } else {
          throw new Error('invalid code')
        }
      } catch (error) {
        console.log(error)
        this.$nuxt.error({ statusCode: 404 })
      } finally {
        this.loading = false
      }
    }
  },
}
</script>

<style lang="scss" scoped>
.qr-code {
  &__container {
    width: auto;
    position: relative;
  }

  &--used > &__img {
    opacity: 0.5;
  }

  &__text {
    display: none;
  }

  &--used > &__text {
    display: block;
    position: absolute;
    font-size: 1.5rem;
    top: 40%;
    // bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    background-color: rgba(255, 255, 255, 0.8);
    padding: 16px 8px;
  }
}
</style>
