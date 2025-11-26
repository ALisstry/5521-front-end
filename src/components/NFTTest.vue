<template>
  <div>
    <div class="card">
      <h2>{{ t('nftTest.title') }}</h2>
      <div class="form-group">
        <label>{{ t('nftTest.metadataUri') }}</label>
        <textarea v-model="tokenURI" rows="3"></textarea>
      </div>
      <button @click="mintNFT">{{ t('nftTest.mint') }}</button>
      <div :class="['status', mintStatusType]" v-if="mintMessage" v-html="mintMessage"></div>
    </div>

    <div class="card">
      <h2>{{ t('nftTest.query') }}</h2>
      <div class="marketplace-actions">
        <button @click="getNFTContractInfo">{{ t('nftTest.getInfo') }}</button>
        <button @click="loadMyNFTs">{{ t('nftTest.loadMyNFTs') }}</button>
        <button @click="getNextTokenId">{{ t('nftTest.getNextId') }}</button>
      </div>
      <div :class="['status', nftStatusType]" v-if="nftMessage" v-html="nftMessage"></div>
    </div>

    <div class="card">
      <h2>{{ t('nftTest.detailed') }}</h2>
      <div class="form-group">
        <label>{{ t('nftTest.tokenId') }}</label>
        <input type="number" v-model="queryTokenId" :placeholder="t('common.tokenIdPlaceholder')" />
      </div>
      <button @click="queryNFTInfo">{{ t('nftTest.queryBtn') }}</button>
      <div :class="['status', queryStatusType]" v-if="queryMessage" v-html="queryMessage"></div>
    </div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'NFTTest',
  props: {
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
      tokenURI: 'https://raw.githubusercontent.com/chiyaqjh/comp5521/main/metadata/1.json',
      mintMessage: '',
      mintStatusType: 'success',
      nftMessage: '',
      nftStatusType: 'success',
      queryTokenId: 0,
      queryMessage: '',
      queryStatusType: 'success',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    async mintNFT() {
      if (!this.nftContract || !web3Service.getAccount()) {
        this.mintMessage = this.t('nftTest.notConnected')
        this.mintStatusType = 'error'
        return
      }

      if (!this.tokenURI) {
        this.mintMessage = this.t('nftTest.invalidUri')
        this.mintStatusType = 'error'
        return
      }

      try {
        this.mintMessage = this.t('nftTest.mintProcessing')
        this.mintStatusType = 'loading'
        this.$emit('debug-info', `Minting NFT, URI: ${this.tokenURI}`)

        const result = await this.nftContract.methods
          .safeMint(web3Service.getAccount(), this.tokenURI)
          .send({
            from: web3Service.getAccount(),
          })

        this.mintMessage = `${this.t('nftTest.mintSuccess')}<br>TX: ${result.transactionHash}`
        this.mintStatusType = 'success'
        this.$emit('debug-info', `Mint tx: ${result.transactionHash}`)

        setTimeout(() => {
          this.getNFTContractInfo()
          this.loadMyNFTs()
        }, 3000)
      } catch (error) {
        this.mintMessage = this.t('nftTest.mintFailed') + error.message
        this.mintStatusType = 'error'
        this.$emit('debug-info', `Mint error: ${error.message}`)
      }
    },

    async getNFTContractInfo() {
      if (!this.nftContract) {
        this.nftMessage = this.t('nftTest.notSet')
        this.nftStatusType = 'error'
        return
      }

      try {
        this.nftMessage = this.t('nftTest.infoProcessing')
        this.nftStatusType = 'loading'

        const name = await this.nftContract.methods.name().call()
        const symbol = await this.nftContract.methods.symbol().call()
        const totalSupply = await this.nftContract.methods.totalSupply().call()
        const nextTokenId = await this.nftContract.methods.getNextTokenId().call()
        const owner = await this.nftContract.methods.owner().call()

        const result = `
          ${this.t('nftTest.infoSuccess')}<br>
          <strong>${this.t('nftTest.contractName')}</strong> ${name}<br>
          <strong>${this.t('nftTest.contractSymbol')}</strong> ${symbol}<br>
          <strong>${this.t('nftTest.contractSupply')}</strong> ${totalSupply}<br>
          <strong>${this.t('nftTest.nextTokenId')}</strong> ${nextTokenId}<br>
          <strong>${this.t('nftTest.contractOwner')}</strong> ${owner}
        `

        this.nftMessage = result
        this.nftStatusType = 'success'
        this.$emit('debug-info', `NFT info: ${name} (${symbol}), supply: ${totalSupply}`)
      } catch (error) {
        this.nftMessage = this.t('nftTest.infoFailed') + error.message
        this.nftStatusType = 'error'
        this.$emit('debug-info', `Get NFT info error: ${error.message}`)
      }
    },

    async loadMyNFTs() {
      if (!this.nftContract || !web3Service.getAccount()) {
        this.nftMessage = this.t('nftTest.notConnected')
        this.nftStatusType = 'error'
        return
      }

      try {
        this.nftMessage = this.t('nftTest.loadingProcessing')
        this.nftStatusType = 'loading'

        const totalSupply = await this.nftContract.methods.totalSupply().call()
        this.$emit('debug-info', `Loading NFTs, supply: ${totalSupply}`)

        let nftsHTML = `<h4>${this.t('nftTest.loadingSuccess')}</h4>`
        let foundNFTs = 0

        for (let i = 0; i < totalSupply; i++) {
          try {
            const owner = await this.nftContract.methods.ownerOf(i).call()
            if (owner.toLowerCase() === web3Service.getAccount().toLowerCase()) {
              const tokenURI = await this.nftContract.methods.tokenURI(i).call()

              let nftImage = ''
              let nftName = `NFT #${i}`

              try {
                const response = await fetch(tokenURI)
                if (response.ok) {
                  const metadata = await response.json()
                  nftImage = metadata.image || ''
                  nftName = metadata.name || nftName
                }
              } catch (e) {
                this.$emit('debug-info', `Get NFT #${i} metadata error: ${e.message}`)
              }

              nftsHTML += `
                <div class="nft-card">
                  ${nftImage ? `<img src="${nftImage}" alt="${nftName}" class="nft-image">` : ''}
                  <div class="nft-info">
                    <div class="nft-title">${nftName}</div>
                    <div class="nft-id">Token ID: ${i}</div>
                    <div class="nft-owner">Owner: ${owner.substring(0, 8)}...${owner.substring(owner.length - 6)}</div>
                    <div class="nft-owner">URI: ${tokenURI}</div>
                  </div>
                </div>
              `
              foundNFTs++
            }
          } catch (error) {
            continue
          }
        }

        if (foundNFTs === 0) {
          nftsHTML += `<p>${this.t('nftTest.noNFTs')}</p>`
        }

        this.nftMessage = nftsHTML
        this.nftStatusType = 'success'
        this.$emit('debug-info', `Found ${foundNFTs} NFTs owned by you`)
      } catch (error) {
        this.nftMessage = this.t('nftTest.loadingFailed') + error.message
        this.nftStatusType = 'error'
        this.$emit('debug-info', `Load NFT error: ${error.message}`)
      }
    },

    async getNextTokenId() {
      if (!this.nftContract) {
        this.nftMessage = this.t('nftTest.notSet')
        this.nftStatusType = 'error'
        return
      }

      try {
        const nextTokenId = await this.nftContract.methods.getNextTokenId().call()
        this.nftMessage = `${this.t('nftTest.querySuccess')}${nextTokenId}`
        this.nftStatusType = 'success'
        this.$emit('debug-info', `Next Token ID: ${nextTokenId}`)
      } catch (error) {
        this.nftMessage = this.t('nftTest.queryFailed') + error.message
        this.nftStatusType = 'error'
        this.$emit('debug-info', `Get next ID error: ${error.message}`)
      }
    },

    async queryNFTInfo() {
      if (!this.nftContract) {
        this.queryMessage = this.t('nftTest.notSet')
        this.queryStatusType = 'error'
        return
      }

      try {
        this.queryMessage = this.t('nftTest.detailedProcessing')
        this.queryStatusType = 'loading'

        const nftInfo = await this.nftContract.methods.getNFTInfo(this.queryTokenId).call()

        let nftImage = ''
        let nftName = `NFT #${this.queryTokenId}`
        let nftDescription = ''

        try {
          const response = await fetch(nftInfo.metadataURI)
          if (response.ok) {
            const metadata = await response.json()
            nftImage = metadata.image || ''
            nftName = metadata.name || nftName
            nftDescription = metadata.description || ''
          }
        } catch (e) {
          this.$emit('debug-info', `Get NFT #${this.queryTokenId} metadata error: ${e.message}`)
        }

        const result = `
          <div class="nft-card">
            ${nftImage ? `<img src="${nftImage}" alt="${nftName}" class="nft-image">` : ''}
            <div class="nft-info">
              <div class="nft-title">${nftName}</div>
              ${nftDescription ? `<div class="nft-id">${nftDescription}</div>` : ''}
              <div class="nft-id">Token ID: ${this.queryTokenId}</div>
              <div class="nft-owner">${this.t('nftTest.currentOwner')}${nftInfo.tokenOwner}</div>
              <div class="nft-owner">${this.t('nftTest.creator')}${nftInfo.creator}</div>
              <div class="nft-owner">${this.t('nftTest.createdTime')}${new Date(nftInfo.createdTime * 1000).toLocaleString()}</div>
              <div class="nft-owner">${this.t('nftTest.metadataUri')}${nftInfo.metadataURI}</div>
              <div class="nft-owner">${this.t('nftTest.totalMinted')}${nftInfo.totalMinted}</div>
            </div>
          </div>
        `

        this.queryMessage = result
        this.queryStatusType = 'success'
        this.$emit('debug-info', `${this.t('nftTest.detailedSuccess')}${this.queryTokenId}`)
      } catch (error) {
        this.queryMessage = this.t('nftTest.detailedFailed') + error.message
        this.queryStatusType = 'error'
        this.$emit('debug-info', `Query NFT error: ${error.message}`)
      }
    },
  },
}
</script>

<style scoped></style>
