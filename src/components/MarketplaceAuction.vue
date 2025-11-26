<template>
  <div class="card">
    <h2>{{ t('marketplaceAuction.title') }}</h2>
    <div class="marketplace-actions">
      <button @click="loadActiveAuctions">{{ t('marketplaceAuction.loadAuctions') }}</button>
      <button @click="finalizeExpiredAuctions">{{ t('marketplaceAuction.finalize') }}</button>
    </div>
    <div class="nft-grid" v-if="auctions.length > 0">
      <div v-for="auction in auctions" :key="auction.tokenId" class="nft-card">
        <img v-if="auction.image" :src="auction.image" :alt="auction.name" class="nft-image" />
        <div class="nft-info">
          <div class="nft-title">{{ auction.name }}</div>
          <div class="nft-id">Token ID: {{ auction.tokenId }}</div>
          <div class="nft-owner">
            {{ t('marketplaceAuction.seller') }} {{ auction.seller.substring(0, 8) }}...{{
              auction.seller.substring(auction.seller.length - 6)
            }}
          </div>
          <div class="auction-info">
            <div><strong>{{ t('marketplaceAuction.currentBid') }}</strong> {{ auction.currentBid }} C5D</div>
            <div><strong>{{ t('marketplaceAuction.nextBid') }}</strong> {{ auction.nextBidAmount }} C5D</div>
            <div><strong>{{ t('marketplaceAuction.timeRemaining') }}</strong> {{ auction.timeRemaining }}</div>
            <div><strong>{{ t('marketplaceAuction.bidIncrement') }}</strong> {{ auction.fixedBidIncrement }} C5D</div>
            <div v-if="auction.hasBids">
              <strong>{{ t('marketplaceAuction.highestBidder') }}</strong> {{ auction.highestBidder.substring(0, 8) }}...{{
                auction.highestBidder.substring(auction.highestBidder.length - 6)
              }}
            </div>
            <div v-else><strong>{{ t('marketplaceAuction.noBids') }}</strong></div>
            <div
              v-if="auction.hasBids"
              class="status error"
              style="margin-top: 10px; padding: 8px; font-size: 12px"
            >
              <strong>{{ t('marketplaceAuction.warning') }}</strong>
            </div>
          </div>
          <div class="marketplace-actions">
            <button class="btn-warning" @click="placeBid(auction.tokenId)">{{ t('marketplaceAuction.bid') }}</button>
          </div>
        </div>
      </div>
    </div>
    <div v-else-if="!loading" class="status">{{ t('marketplaceAuction.noAuctions') }}</div>
    <div v-else class="status loading">{{ t('common.loading') }}</div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { formatTimeRemaining } from '../utils/helpers'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'MarketplaceAuction',
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
      auctions: [],
      loading: false,
      currentAccount: web3Service.getAccount() || '',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    async loadActiveAuctions() {
      if (!this.marketplaceContract) {
        alert(this.t('marketplaceAuction.notSet'))
        return
      }

      this.loading = true
      try {
        const result = await this.marketplaceContract.methods.getAllActiveAuctions().call()

        const tokenIds = result[0].map((id) => Number(id))
        const sellers = result[1]
        const currentBids = result[2].map((bid) => Number(bid))
        const timeRemaining = result[3].map((time) => Number(time))
        const finalizedStatus = result[4]

        this.auctions = []

        for (let i = 0; i < tokenIds.length; i++) {
          if (finalizedStatus[i]) continue

          const tokenId = tokenIds[i]
          const seller = sellers[i]
          const currentBid = currentBids[i]
          const remainingTime = timeRemaining[i]

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

          let auctionStatus
          try {
            auctionStatus = await this.marketplaceContract.methods.getAuctionStatus(tokenId).call()
          } catch (e) {
            auctionStatus = [
              false,
              0,
              '0x0000000000000000000000000000000000000000',
              0,
              0,
              0,
              0,
              0,
              false,
            ]
          }

          const highestBidder = auctionStatus[2]
          const highestBid = Number(auctionStatus[3])
          const nextBidAmount = Number(auctionStatus[4])
          const fixedBidIncrement = Number(auctionStatus[5])

          const hasBids =
            highestBidder !== '0x0000000000000000000000000000000000000000' && highestBid > 0

          this.auctions.push({
            tokenId,
            seller,
            currentBid,
            timeRemaining: formatTimeRemaining(remainingTime),
            name: nftName,
            image: nftImage,
            highestBidder,
            nextBidAmount,
            fixedBidIncrement,
            hasBids,
          })
        }

        this.$emit('debug-info', `Loaded ${this.auctions.length} auctions`)
      } catch (error) {
        alert(this.t('marketplaceAuction.loadingFailed') + error.message)
        this.$emit('debug-info', `Load auctions error: ${error.message}`)
      } finally {
        this.loading = false
      }
    },

    async placeBid(tokenId) {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert(this.t('marketplaceAuction.notConnected'))
        return
      }

      try {
        const result = await this.marketplaceContract.methods.placeBid(tokenId).send({
          from: web3Service.getAccount(),
        })

        alert(`${this.t('marketplaceAuction.bidSuccess')}${result.transactionHash}`)
        this.$emit('debug-info', `Bid NFT #${tokenId} tx: ${result.transactionHash}`)

        this.loadActiveAuctions()
      } catch (error) {
        alert(this.t('marketplaceAuction.bidFailed') + error.message)
        this.$emit('debug-info', `Bid error: ${error.message}`)
      }
    },

    async finalizeExpiredAuctions() {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert(this.t('marketplaceAuction.notConnected'))
        return
      }

      try {
        const result = await this.marketplaceContract.methods.finalize_Expired_Auctions().send({
          from: web3Service.getAccount(),
        })

        alert(`${this.t('marketplaceAuction.finalizeSuccess')}${result.transactionHash}`)
        this.$emit('debug-info', `Finalize auctions tx: ${result.transactionHash}`)

        this.loadActiveAuctions()
      } catch (error) {
        alert(this.t('marketplaceAuction.finalizeFailed') + error.message)
        this.$emit('debug-info', `Finalize error: ${error.message}`)
      }
    },
  },
}
</script>

<style scoped></style>
