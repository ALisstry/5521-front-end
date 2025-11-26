<template>
  <div class="card">
    <h2>{{ t('contractSetup.title') }}</h2>
    <div class="contract-setup">
      <div class="contract-item">
        <h3>{{ t('contractSetup.stablecoin.title') }}</h3>
        <input
          v-model="stablecoinAddr"
          type="text"
          :placeholder="t('common.placeholderAddress')"
          class="contract-address"
        />
        <button @click="setStablecoinContract">{{ t('contractSetup.stablecoin.button') }}</button>
        <div :class="['status', stablecoinStatusType]" v-if="stablecoinStatusMessage">
          {{ stablecoinStatusMessage }}
        </div>
      </div>

      <div class="contract-item">
        <h3>{{ t('contractSetup.nft.title') }}</h3>
        <input
          v-model="nftAddr"
          type="text"
          :placeholder="t('common.placeholderAddress')"
          class="contract-address"
        />
        <button @click="setNFTContract">{{ t('contractSetup.nft.button') }}</button>
        <div :class="['status', nftStatusType]" v-if="nftStatusMessage">{{ nftStatusMessage }}</div>
      </div>

      <div class="contract-item">
        <h3>{{ t('contractSetup.marketplace.title') }}</h3>
        <input
          v-model="marketplaceAddr"
          type="text"
          :placeholder="t('common.placeholderAddress')"
          class="contract-address"
        />
        <button @click="setMarketplaceContract">{{ t('contractSetup.marketplace.button') }}</button>
        <div :class="['status', marketplaceStatusType]" v-if="marketplaceStatusMessage">
          {{ marketplaceStatusMessage }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import contractService from '../services/contractService'
import { isValidAddress } from '../utils/helpers'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'ContractSetup',
  props: {
    language: {
      type: String,
      default: 'en',
    },
  },
  data() {
    return {
      stablecoinAddr: '',
      nftAddr: '',
      marketplaceAddr: '',
      stablecoinStatusMessage: '',
      stablecoinStatusType: 'success',
      nftStatusMessage: '',
      nftStatusType: 'success',
      marketplaceStatusMessage: '',
      marketplaceStatusType: 'success',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    setStablecoinContract() {
      const address = this.stablecoinAddr.trim()

      if (!web3Service.web3) {
        this.stablecoinStatusMessage = this.t('contractSetup.stablecoin.notConnected')
        this.stablecoinStatusType = 'error'
        return
      }

      if (!isValidAddress(address)) {
        this.stablecoinStatusMessage = this.t('contractSetup.stablecoin.invalid')
        this.stablecoinStatusType = 'error'
        return
      }

      try {
        contractService.setWeb3(web3Service.web3)
        const contract = contractService.setStablecoinContract(address)
        this.stablecoinStatusMessage = this.t('contractSetup.stablecoin.success')
        this.stablecoinStatusType = 'success'
        this.$emit('debug-info', `Stablecoin contract address: ${address}`)
        this.$emit('stablecoin-contract-set', contract)
        this.testStablecoinConnection(contract)
      } catch (error) {
        this.stablecoinStatusMessage = this.t('contractSetup.stablecoin.failed') + error.message
        this.stablecoinStatusType = 'error'
        this.$emit('debug-info', `Stablecoin contract error: ${error.message}`)
      }
    },

    setNFTContract() {
      const address = this.nftAddr.trim()

      if (!web3Service.web3) {
        this.nftStatusMessage = this.t('contractSetup.nft.notConnected')
        this.nftStatusType = 'error'
        return
      }

      if (!isValidAddress(address)) {
        this.nftStatusMessage = this.t('contractSetup.nft.invalid')
        this.nftStatusType = 'error'
        return
      }

      try {
        contractService.setWeb3(web3Service.web3)
        const contract = contractService.setNFTContract(address)
        this.nftStatusMessage = this.t('contractSetup.nft.success')
        this.nftStatusType = 'success'
        this.$emit('debug-info', `NFT contract address: ${address}`)
        this.$emit('nft-contract-set', contract)
        this.testNFTConnection(contract)
      } catch (error) {
        this.nftStatusMessage = this.t('contractSetup.nft.failed') + error.message
        this.nftStatusType = 'error'
        this.$emit('debug-info', `NFT contract error: ${error.message}`)
      }
    },

    setMarketplaceContract() {
      const address = this.marketplaceAddr.trim()

      if (!web3Service.web3) {
        this.marketplaceStatusMessage = this.t('contractSetup.marketplace.notConnected')
        this.marketplaceStatusType = 'error'
        return
      }

      if (!isValidAddress(address)) {
        this.marketplaceStatusMessage = this.t('contractSetup.marketplace.invalid')
        this.marketplaceStatusType = 'error'
        return
      }

      try {
        contractService.setWeb3(web3Service.web3)
        const contract = contractService.setMarketplaceContract(address)
        this.marketplaceStatusMessage = this.t('contractSetup.marketplace.success')
        this.marketplaceStatusType = 'success'
        this.$emit('debug-info', `Marketplace contract address: ${address}`)
        this.$emit('marketplace-contract-set', contract)
      } catch (error) {
        this.marketplaceStatusMessage = this.t('contractSetup.marketplace.failed') + error.message
        this.marketplaceStatusType = 'error'
        this.$emit('debug-info', `Marketplace contract error: ${error.message}`)
      }
    },

    async testStablecoinConnection(contract) {
      try {
        this.$emit('debug-info', this.t('contractSetup.stablecoin.testingConnection'))
        const name = await contract.methods.name().call()
        const symbol = await contract.methods.symbol().call()
        this.$emit('debug-info', `Stablecoin: ${name}, Symbol: ${symbol}`)
      } catch (error) {
        this.$emit(
          'debug-info',
          this.t('contractSetup.stablecoin.connectionFailed') + error.message,
        )
      }
    },

    async testNFTConnection(contract) {
      try {
        this.$emit('debug-info', this.t('contractSetup.nft.testingConnection'))
        const name = await contract.methods.name().call()
        const symbol = await contract.methods.symbol().call()
        const owner = await contract.methods.owner().call()
        this.$emit('debug-info', `NFT: ${name}, Symbol: ${symbol}, Owner: ${owner}`)
      } catch (error) {
        this.$emit('debug-info', this.t('contractSetup.nft.connectionFailed') + error.message)
      }
    },
  },
}
</script>

<style scoped></style>
