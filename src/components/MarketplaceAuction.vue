<template>
  <div class="card">
    <h2>⚡ 市场拍卖</h2>
    <div class="marketplace-actions">
      <button @click="loadActiveAuctions">🔄 加载活跃拍卖</button>
      <button @click="finalizeExpiredAuctions">⏰ 结束过期拍卖</button>
    </div>
    <div class="nft-grid" v-if="auctions.length > 0">
      <div v-for="auction in auctions" :key="auction.tokenId" class="nft-card">
        <img v-if="auction.image" :src="auction.image" :alt="auction.name" class="nft-image" />
        <div class="nft-info">
          <div class="nft-title">{{ auction.name }}</div>
          <div class="nft-id">Token ID: {{ auction.tokenId }}</div>
          <div class="nft-owner">
            卖家: {{ auction.seller.substring(0, 8) }}...{{
              auction.seller.substring(auction.seller.length - 6)
            }}
          </div>
          <div class="auction-info">
            <div><strong>当前最高出价:</strong> {{ auction.currentBid }} C5D</div>
            <div><strong>下一次出价:</strong> {{ auction.nextBidAmount }} C5D</div>
            <div><strong>剩余时间:</strong> {{ auction.timeRemaining }}</div>
            <div><strong>固定加价:</strong> {{ auction.fixedBidIncrement }} C5D</div>
            <div v-if="auction.hasBids">
              <strong>最高出价者:</strong> {{ auction.highestBidder.substring(0, 8) }}...{{
                auction.highestBidder.substring(auction.highestBidder.length - 6)
              }}
            </div>
            <div v-else><strong>状态:</strong> 暂无出价</div>
            <div
              v-if="auction.hasBids"
              class="status error"
              style="margin-top: 10px; padding: 8px; font-size: 12px"
            >
              <strong>⚠️ 注意:</strong> 已有出价，拍卖无法取消
            </div>
          </div>
          <div class="marketplace-actions">
            <button class="btn-warning" @click="placeBid(auction.tokenId)">出价</button>
          </div>
        </div>
      </div>
    </div>
    <div v-else-if="!loading" class="status">暂无活跃拍卖</div>
    <div v-else class="status loading">🔄 加载中...</div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { formatTimeRemaining } from '../utils/helpers'

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
  },
  data() {
    return {
      auctions: [],
      loading: false,
      currentAccount: web3Service.getAccount() || '',
    }
  },
  methods: {
    async loadActiveAuctions() {
      if (!this.marketplaceContract) {
        alert('请先设置市场合约地址')
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
            this.$emit('debug-info', `获取NFT #${tokenId} 元数据失败: ${e.message}`)
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

        this.$emit('debug-info', `加载了 ${this.auctions.length} 个活跃拍卖`)
      } catch (error) {
        alert('加载活跃拍卖失败: ' + error.message)
        this.$emit('debug-info', `加载活跃拍卖错误: ${error.message}`)
      } finally {
        this.loading = false
      }
    },

    async placeBid(tokenId) {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert('请先连接钱包并设置市场合约')
        return
      }

      try {
        const result = await this.marketplaceContract.methods.placeBid(tokenId).send({
          from: web3Service.getAccount(),
        })

        alert(`出价成功！交易哈希: ${result.transactionHash}`)
        this.$emit('debug-info', `对NFT #${tokenId} 出价交易: ${result.transactionHash}`)

        this.loadActiveAuctions()
      } catch (error) {
        alert(`出价失败: ${error.message}`)
        this.$emit('debug-info', `对NFT #${tokenId} 出价错误: ${error.message}`)
      }
    },

    async finalizeExpiredAuctions() {
      if (!this.marketplaceContract || !web3Service.getAccount()) {
        alert('请先连接钱包并设置市场合约')
        return
      }

      try {
        const result = await this.marketplaceContract.methods.finalize_Expired_Auctions().send({
          from: web3Service.getAccount(),
        })

        alert(`结束过期拍卖成功！交易哈希: ${result.transactionHash}`)
        this.$emit('debug-info', `结束过期拍卖交易: ${result.transactionHash}`)

        this.loadActiveAuctions()
      } catch (error) {
        alert(`结束过期拍卖失败: ${error.message}`)
        this.$emit('debug-info', `结束过期拍卖错误: ${error.message}`)
      }
    },
  },
}
</script>

<style scoped></style>
