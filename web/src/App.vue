<template>
  <div id="app">
    <Navigation></Navigation>
    <keep-alive include="HomePage,MarketsPage,OrdersPage,BalancesPage">
      <router-view></router-view>
    </keep-alive>
    <AlertBar :btcUsdMarket="btcUsdMarket"></AlertBar>
    <div class="footer" v-if="isAuthorized">
      <Button :label="'Logout'" :className="'link-gray'" @click.native="handleLogout()"></Button> &nbsp;
      <Button :label="'Setup'" :className="'link-gray'" @click.native="handleSetup()"></Button>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Navigation from '@/components/Navigation'
import Button from '@/components/Button'
import AlertBar from '@/components/AlertBar'

export default {
  name: 'app',
  components: {
    Navigation,
    Button,
    AlertBar
  },
  computed: {
    ...mapGetters({
      isAuthorized: 'auth/isAuthorized',
      btcUsdMarket: 'markets/btcUsdMarket'
    })
  },
  created () {
    if (this.isAuthorized) {
      this.$store.commit('exchanges/setSelected', 'bittrex')
      this.getAllData()
    }
  },
  methods: {
    handleLogout () {
      this.$store.commit('auth/removeToken')
      window.clearInterval(this.marketInterval)
      window.location.reload()
    },
    handleSetup () {
      this.$router.push('/setup')
    },
    getAllData () {
      // Then we get all market data. We use this throughout the whole website, so we want this to be available after first load
      this.$store.dispatch('markets/getAll')
      this.$store.dispatch('balances/getAll')

      // TODO: do with websockets
      this.marketInterval = setInterval(() => {
        this.$store.dispatch('markets/getAll')
      }, 2000)
    }
  },
  watch: {
    isAuthorized: function (newValue) {
      if (newValue === true) {
        this.getAllData()
      }
    }
  }
}
</script>

<style lang="scss">
html {
  font-size: 62.5%;
  background: $color-athens-gray;
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}

body {
  padding: 0;
  margin: 0;
  font-size: 1.4rem;
  line-height: 2.4rem;
}

#app {
  max-width: $max-page-width;
  margin: 0 auto;
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: $color-pickled-bluewood;
  padding-top: 76px;
}

label {
  font-weight: bold;
  display: block;
  line-height: 2.4rem;
}

input[type=text],
input[type=number] {
  border-radius: 3px;
  font-size: 1.4rem;
  height: 4rem;
  padding: 0 15px;
  line-height: 4rem;
  border: 1px $color-loblolly solid;
  width: 100%;
  appearance: none;

  &:focus {
    border-color: $color-azure-radiance;
  }
}

a {
  color: $color-azure-radiance;
}

.footer {
  text-align: center;
  margin: 50px 0;
}

</style>
