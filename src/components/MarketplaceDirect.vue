<template>
  <div class="card">
    <h2>{{ t('marketplaceDirect.title') }}</h2>
    <button @click="loadFixedPriceListings">{{ t('marketplaceDirect.loadListings') }}</button>
    <div class="nft-grid" v-if="listings.length > 0">
      <div v-for="listing in listings" :key="listing.tokenId" class="nft-card">
        <img v-if="listing.image" :src="listing.image" :alt="listing.name" class="nft-image" />
        <div class="nft-info">
          <div class="nft-title">{{ listing.name }}</div>
          <div class="nft-id">Token ID: {{ listing.tokenId }}</div>
          <div class="nft-owner">
            {{ t('marketplaceDirect.seller') }} {{ listing.seller.substring(0, 8) }}...{{
              listing.seller.substring(listing.seller.length - 6)
            }}
          </div>
          <div class="nft-price">{{ t('marketplaceDirect.price') }}{{ listing.price }} C5D</div>
          <div class="marketplace-actions">
            <button class="btn-success" @click="buyNFT(listing.tokenId)">{{ t('marketplaceDirect.buyer') }}</button>
            <button
              v-if="listing.seller.toLowerCase() === currentAccount.toLowerCase()"
              class="btn-danger"
              @click="cancelListing(listing.tokenId)"
            >
              {{ t('marketplaceDirect.cancel') }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-else-if="!loading" class="status">{{ t('marketplaceDirect.noListings') }}</div>
    <div v-else class="status loading">{{ t('common.loading') }}</div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'MarketplaceDirect',
  props: {
    marketplaceContract: {
      type: Object,
      default: null,
    },
    nftContract: {
      type: Object,
      default: null,
    },
    language: {
      type: String,
      default: 'en',
    },
  },
  data() {
    return {
      listings: [],
      loading: false,
      currentAccount: web3Service.getAccount() || '',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    async loadFixedPriceListings() {
      if (!this.marketplaceContract) {
        alert(this.t('marketplaceDirect.notSet'))
        return
      }

      this.loading = true
      try {
        const result = await this.marketplaceContract.methods.getAllActiveListings().call()

        const tokenIds = result[0].map((id) => Number(id))
        const sellers = result[1]
        const prices = result[2].map((price) => Number(price))
        const activeStatus = result[3]

        this.listings = []

        for (let i = 0; i < tokenIds.length; i++) {
          if (!activeStatus[i]) continue

          const tokenId = tokenIds[i]
          const seller = sellers[i]
          const price = prices[i]

          let nftImage = ''
          let nftName = `NFT #${tokenId}`

          try {
            const tokenURI = await this.nftContract.methods.tokenURI(tokenId).call()
            const response = await fetch(tokenURI)
            if (response.ok) {
              const metadata = await response.json()
              nftImage = metadata.image || ''
              nftName = metadata.name || nftName
            }
          } catch (e) {
            this.$emit('debug-info', `Get NFT #${tokenId} metadata error: ${e.message}`)
          }

          this.listings.push({
            tokenId,
            seller,
            price,
            name: nftName,
            image: nftImage,
          })
        }

        this.$emit('debug-info', `Loaded ${this.listings.length} listings`)
      } catch (error) {
        alert(this.t('marketplaceDirect.loadingFailed') + error.message)
        this.$emit('debug-info', `Load listings error: ${error.message}`)
      } finally {
        this.loading = false
      }
    },

    async buyNFT(tokenId) {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert(this.t('marketplaceDirect.notConnected'))
        return
      }

      try {
        const result = await this.marketplaceContract.methods.buyNFT(tokenId).send({
          from: web3Service.getAccount(),
        })

        alert(`${this.t('marketplaceDirect.buySuccess')}${result.transactionHash}`)
        this.$emit('debug-info', `Buy NFT #${tokenId} tx: ${result.transactionHash}`)

        this.loadFixedPriceListings()
      } catch (error) {
        alert(this.t('marketplaceDirect.buyFailed') + error.message)
        this.$emit('debug-info', `Buy error: ${error.message}`)
      }
    },

    async cancelListing(tokenId) {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert(this.t('marketplaceDirect.notConnected'))
        return
      }

      try {
        const result = await this.marketplaceContract.methods.cancelListing(tokenId).send({
          from: web3Service.getAccount(),
        })

        alert(`${this.t('marketplaceDirect.cancelSuccess')}${result.transactionHash}`)
        this.$emit('debug-info', `Cancel listing #${tokenId} tx: ${result.transactionHash}`)

        this.loadFixedPriceListings()
      } catch (error) {
        alert(this.t('marketplaceDirect.cancelFailed') + error.message)
        this.$emit('debug-info', `Cancel error: ${error.message}`)
      }
    },
  },
}
</script>

<style scoped></style>
