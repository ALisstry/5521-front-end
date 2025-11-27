<template>
  <div class="container">
    <aside class="sidebar">
      <div class="brand">
        <div class="logo">NFT</div>
        <h3>{{ t('title') }}</h3>
      </div>
      <div class="small">{{ t('subtitle') }}</div>

      <div class="nav">
        <div
          v-for="tab in tabs"
          :key="tab.id"
          :class="['nav-btn', { active: activeTab === tab.id }]"
          @click="activeTab = tab.id"
        >
          {{ tabLabel(tab.id) }}
        </div>
      </div>

      <div style="margin-top: 16px">
        <LanguageSwitcher v-model="language" />
      </div>
    </aside>

    <main class="main">
      <div class="header">
        <div class="title">
          <h1>{{ t('title') }}</h1>
          <p>{{ t('subtitle') }}</p>
        </div>
        <div class="actions">
          <WalletConnection
            :language="language"
            @wallet-connected="onWalletConnected"
            @wallet-error="onWalletError"
            @debug-info="addDebugInfo"
          />
        </div>
      </div>

      <ContractSetup
        :language="language"
        @stablecoin-contract-set="onStablecoinContractSet"
        @nft-contract-set="onNFTContractSet"
        @marketplace-contract-set="onMarketplaceContractSet"
        @debug-info="addDebugInfo"
      />

      <div class="content">
        <div v-if="activeTab === 'stablecoin'">
          <StablecoinTest
            :stablecoin-contract="stablecoinContract"
            :language="language"
            @debug-info="addDebugInfo"
          />
        </div>

        <div v-if="activeTab === 'nft'">
          <NFTTest :nft-contract="nftContract" :language="language" @debug-info="addDebugInfo" />
        </div>

        <div v-if="activeTab === 'marketplace'">
          <MarketplaceDirect
            :marketplace-contract="marketplaceContract"
            :nft-contract="nftContract"
            :language="language"
            @debug-info="addDebugInfo"
          />
        </div>

        <div v-if="activeTab === 'auction'">
          <MarketplaceAuction
            :marketplace-contract="marketplaceContract"
            :nft-contract="nftContract"
            :language="language"
            @debug-info="addDebugInfo"
          />
        </div>

        <div v-if="activeTab === 'debug'">
          <DebugInfo :language="language" ref="debugComponent" />
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import WalletConnection from './components/WalletConnection.vue'
import ContractSetup from './components/ContractSetup.vue'
import StablecoinTest from './components/StablecoinTest.vue'
import NFTTest from './components/NFTTest.vue'
import MarketplaceDirect from './components/MarketplaceDirect.vue'
import MarketplaceAuction from './components/MarketplaceAuction.vue'
import DebugInfo from './components/DebugInfo.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import { getTranslation } from './utils/i18n.js'

export default {
  name: 'App',
  components: {
    WalletConnection,
    ContractSetup,
    StablecoinTest,
    NFTTest,
    MarketplaceDirect,
    MarketplaceAuction,
    DebugInfo,
    LanguageSwitcher,
  },
  data() {
    return {
      language: 'en',
      activeTab: 'stablecoin',
      tabs: [
        { id: 'stablecoin' },
        { id: 'nft' },
        { id: 'marketplace' },
        { id: 'auction' },
        // { id: 'debug' },
      ],
      stablecoinContract: null,
      nftContract: null,
      marketplaceContract: null,
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    tabLabel(tabId) {
      return getTranslation(this.language, `tabs.${tabId}`)
    },
    onWalletConnected(data) {
      this.addDebugInfo(
        `${this.t('walletConnection.success')} ${data.account}, ChainID: ${data.chainId}`,
      )
    },
    onWalletError(error) {
      this.addDebugInfo(`${this.t('walletConnection.failed')} ${error.message}`)
    },
    onStablecoinContractSet(contract) {
      this.stablecoinContract = contract
      this.addDebugInfo(`${this.t('contractSetup.stablecoin.success')}`)
    },
    onNFTContractSet(contract) {
      this.nftContract = contract
      this.addDebugInfo(`${this.t('contractSetup.nft.success')}`)
    },
    onMarketplaceContractSet(contract) {
      this.marketplaceContract = contract
      this.addDebugInfo(`${this.t('contractSetup.marketplace.success')}`)
    },
    addDebugInfo(message) {
      if (this.$refs.debugComponent) {
        this.$refs.debugComponent.addDebugInfo(message)
      }
    },
  },
  mounted() {
    const savedLanguage = localStorage.getItem('language')
    if (savedLanguage) {
      this.language = savedLanguage
    }
    this.addDebugInfo('Application loaded')
  },
}
</script>

<style scoped></style>
