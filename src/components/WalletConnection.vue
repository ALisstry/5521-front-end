<template>
  <div class="card">
    <h2>🔗 连接钱包</h2>
    <button @click="connectMetaMask" :disabled="isConnected">
      {{ isConnected ? '已连接' : '连接 MetaMask' }}
    </button>
    <div :class="['status', statusType]" v-if="statusMessage">{{ statusMessage }}</div>

    <div v-if="isConnected" id="walletInfo" class="wallet-info">
      <div class="wallet-info-item">
        <strong>钱包地址</strong>
        <span class="contract-address">{{ accountAddress }}</span>
      </div>
      <div class="wallet-info-item">
        <strong>网络</strong>
        <span>{{ networkName }}</span>
      </div>
      <div class="wallet-info-item">
        <strong>链ID</strong>
        <span>{{ chainId }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import { getNetworkName } from '../utils/helpers'

export default {
  name: 'WalletConnection',
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
  mounted() {
    this.initWallet()
  },
  methods: {
    initWallet() {
      if (!web3Service.isMetaMaskAvailable()) {
        this.statusMessage = '❌ 未检测到 MetaMask，请先安装 MetaMask 浏览器扩展'
        this.statusType = 'error'
        return
      }
      this.statusMessage = '✅ 已检测到 MetaMask，点击上方按钮连接'
      this.statusType = 'success'

      // 监听账户变化
      web3Service.onAccountChanged((accounts) => {
        if (accounts.length === 0) {
          this.isConnected = false
          this.statusMessage = '🔒 钱包已断开连接'
          this.statusType = 'error'
        } else {
          this.accountAddress = accounts[0]
          this.$emit('account-changed', accounts[0])
        }
      })

      // 监听网络变化
      web3Service.onChainChanged((chainId) => {
        this.chainId = chainId
        this.networkName = getNetworkName(chainId)
        this.$emit('chain-changed', chainId)
      })
    },
    async connectMetaMask() {
      if (this.isConnected) return

      try {
        this.statusMessage = '🔄 正在连接 MetaMask...'
        this.statusType = 'loading'

        await web3Service.initWeb3()
        const result = await web3Service.connectWallet()

        this.isConnected = true
        this.accountAddress = result.account
        this.chainId = result.chainId
        this.networkName = getNetworkName(result.chainId)

        this.statusMessage = '✅ MetaMask 连接成功！'
        this.statusType = 'success'

        this.$emit('wallet-connected', {
          account: result.account,
          chainId: result.chainId,
        })
      } catch (error) {
        this.isConnected = false
        if (error.code === 4001) {
          this.statusMessage = '❌ 用户拒绝了连接请求'
        } else {
          this.statusMessage = '❌ 连接失败: ' + error.message
        }
        this.statusType = 'error'
        this.$emit('wallet-error', error)
      }
    },
  },
}
</script>

<style scoped></style>
