<template>
  <div class="card">
    <h2>{{ t('walletConnection.title') }}</h2>
    <button @click="connectMetaMask" :disabled="isConnected">
      {{ isConnected ? t('walletConnection.connected') : t('walletConnection.button') }}
    </button>
    <div :class="['status', statusType]" v-if="statusMessage">{{ statusMessage }}</div>

    <div v-if="isConnected" id="walletInfo" class="wallet-info">
      <div class="wallet-info-item">
        <strong>{{ t('walletConnection.walletAddress') }}</strong>
        <span class="contract-address">{{ accountAddress }}</span>
      </div>
      <div class="wallet-info-item">
        <strong>{{ t('walletConnection.network') }}</strong>
        <span>{{ networkName }}</span>
      </div>
      <div class="wallet-info-item">
        <strong>{{ t('walletConnection.chainId') }}</strong>
        <span>{{ chainId }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { getNetworkName } from '../utils/helpers'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'WalletConnection',
  props: {
    language: {
      type: String,
      default: 'en',
    },
  },
  data() {
    return {
      isConnected: false,
      accountAddress: '',
      chainId: '',
      networkName: '',
      statusMessage: '',
      statusType: 'success',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    initWallet() {
      if (!web3Service.isMetaMaskAvailable()) {
        this.statusMessage = this.t('walletConnection.notDetected')
        this.statusType = 'error'
        return
      }
      this.statusMessage = this.t('walletConnection.detected')
      this.statusType = 'success'

      web3Service.onAccountChanged((accounts) => {
        if (accounts.length === 0) {
          this.isConnected = false
          this.statusMessage = this.t('walletConnection.disconnected')
          this.statusType = 'error'
        } else {
          this.accountAddress = accounts[0]
          this.$emit('account-changed', accounts[0])
        }
      })

      web3Service.onChainChanged((chainId) => {
        this.chainId = chainId
        this.networkName = getNetworkName(chainId)
        this.$emit('chain-changed', chainId)
      })
    },
    async connectMetaMask() {
      if (this.isConnected) return

      try {
        this.statusMessage = this.t('common.loading') + this.t('walletConnection.button')
        this.statusType = 'loading'

        await web3Service.initWeb3()
        const result = await web3Service.connectWallet()

        this.isConnected = true
        this.accountAddress = result.account
        this.chainId = result.chainId
        this.networkName = getNetworkName(result.chainId)

        this.statusMessage = this.t('walletConnection.success')
        this.statusType = 'success'

        this.$emit('wallet-connected', {
          account: result.account,
          chainId: result.chainId,
        })
      } catch (error) {
        this.isConnected = false
        if (error.code === 4001) {
          this.statusMessage = this.t('walletConnection.denied')
        } else {
          this.statusMessage = this.t('walletConnection.failed') + error.message
        }
        this.statusType = 'error'
        this.$emit('wallet-error', error)
      }
    },
  },
  mounted() {
    this.initWallet()
  },
}
</script>

<style scoped></style>
