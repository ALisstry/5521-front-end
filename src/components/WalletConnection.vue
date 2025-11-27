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
      // track message key so it can be recomputed when language changes
      statusKey: null,
      statusMeta: null,
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    setStatus(key, type = 'success', meta = null) {
      this.statusKey = key
      this.statusType = type
      this.statusMeta = meta
      const base = key ? this.t(`walletConnection.${key}`) : ''
      this.statusMessage = base + (meta ? meta : '')
    },
    initWallet() {
      if (!web3Service.isMetaMaskAvailable()) {
        this.setStatus('notDetected', 'error')
        return
      }
      this.setStatus('detected', 'success')

      web3Service.onAccountChanged((accounts) => {
        if (accounts.length === 0) {
          this.isConnected = false
          this.setStatus('disconnected', 'error')
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
        // show loading with button label; keep meta so it can be recomputed
        this.setStatus(
          null,
          'loading',
          this.t('common.loading') + this.t('walletConnection.button'),
        )

        await web3Service.initWeb3()
        const result = await web3Service.connectWallet()

        this.isConnected = true
        this.accountAddress = result.account
        this.chainId = result.chainId
        this.networkName = getNetworkName(result.chainId)

        this.setStatus('success', 'success')

        this.$emit('wallet-connected', {
          account: result.account,
          chainId: result.chainId,
        })
      } catch (error) {
        this.isConnected = false
        if (error.code === 4001) {
          this.setStatus('denied', 'error')
        } else {
          this.setStatus('failed', 'error', error.message)
        }
        this.$emit('wallet-error', error)
      }
    },
  },
  watch: {
    language(newLang) {
      // recompute statusMessage when language changes
      if (this.statusKey || this.statusMeta) {
        const meta = this.statusMeta
        if (this.statusKey) {
          this.statusMessage = this.t(`walletConnection.${this.statusKey}`) + (meta ? meta : '')
        } else {
          // statusKey is null but there may be meta (like loading + button label)
          this.statusMessage = meta ? meta : ''
        }
      }
    },
  },
  mounted() {
    this.initWallet()
  },
}
</script>

<style scoped></style>
