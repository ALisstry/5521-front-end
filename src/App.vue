<template>
  <div class="container">
    <!-- 语言切换器 -->
    <LanguageSwitcher v-model="language" />

    <!-- 页面头部 -->
    <header>
      <h1>{{ t('title') }}</h1>
      <p>{{ t('subtitle') }}</p>
    </header>

    <!-- 钱包连接区域 -->
    <WalletConnection
      :language="language"
      @wallet-connected="onWalletConnected"
      @wallet-error="onWalletError"
      @debug-info="addDebugInfo"
    />

    <!-- 合约地址设置 -->
    <ContractSetup
      :language="language"
      @stablecoin-contract-set="onStablecoinContractSet"
      @nft-contract-set="onNFTContractSet"
      @marketplace-contract-set="onMarketplaceContractSet"
      @debug-info="addDebugInfo"
    />

    <!-- 标签页导航 -->
    <div class="tabs">
      <div
        v-for="tab in tabs"
        :key="tab.id"
        :class="['tab', { active: activeTab === tab.id }]"
        @click="activeTab = tab.id"
      >
        {{ tabLabel(tab.id) }}
      </div>
    </div>

    <!-- 稳定币测试标签页 -->
    <div v-if="activeTab === 'stablecoin'" class="tab-content active">
      <StablecoinTest
        :stablecoin-contract="stablecoinContract"
        :language="language"
        @debug-info="addDebugInfo"
      />
    </div>

    <!-- NFT测试标签页 -->
    <div v-if="activeTab === 'nft'" class="tab-content active">
      <NFTTest :nft-contract="nftContract" :language="language" @debug-info="addDebugInfo" />
    </div>

    <!-- 市场直接购买标签页 -->
    <div v-if="activeTab === 'marketplace'" class="tab-content active">
      <MarketplaceDirect
        :marketplace-contract="marketplaceContract"
        :nft-contract="nftContract"
        :language="language"
        @debug-info="addDebugInfo"
      />
    </div>

    <!-- 市场拍卖标签页 -->
    <div v-if="activeTab === 'auction'" class="tab-content active">
      <MarketplaceAuction
        :marketplace-contract="marketplaceContract"
        :nft-contract="nftContract"
        :language="language"
        @debug-info="addDebugInfo"
      />
    </div>

    <!-- 调试信息标签页 -->
    <div v-if="activeTab === 'debug'" class="tab-content active">
      <DebugInfo :language="language" ref="debugComponent" />
    </div>
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
        { id: 'debug' },
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
