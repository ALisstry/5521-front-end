<template>
  <div class="card">
    <h2>💰 稳定币功能测试</h2>
    <div class="marketplace-actions">
      <button @click="getTokenInfo">🔍 获取代币信息</button>
      <button @click="useFaucet">🚰 领取测试代币</button>
      <button @click="checkBalance">💰 查询余额</button>
    </div>
    <div :class="['status', resultType]" v-if="resultMessage" v-html="resultMessage"></div>
  </div>
</template>

<script>
import web3Service from '../services/web3Service'
import contractService from '../services/contractService'

export default {
  name: 'StablecoinTest',
  props: {
    stablecoinContract: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      resultMessage: '',
      resultType: 'success',
    }
  },
  methods: {
    async getTokenInfo() {
      if (!this.stablecoinContract) {
        this.resultMessage = '❌ 请先设置稳定币合约地址'
        this.resultType = 'error'
        return
      }

      try {
        this.resultMessage = '🔄 获取代币信息中...'
        this.resultType = 'loading'

        const tokenInfo = await this.stablecoinContract.methods.getTokenInfo().call()

        const result = `
          ✅ 代币信息获取成功！<br>
          <strong>名称:</strong> ${tokenInfo.tokenName}<br>
          <strong>符号:</strong> ${tokenInfo.tokenSymbol}<br>
          <strong>总供应量:</strong> ${web3Service.web3.utils.fromWei(tokenInfo.tokenTotalSupply, 'ether')} ${tokenInfo.tokenSymbol}<br>
          <strong>合约所有者:</strong> ${tokenInfo.tokenOwner}
        `

        this.resultMessage = result
        this.resultType = 'success'
        this.$emit('debug-info', `代币信息: ${tokenInfo.tokenName} (${tokenInfo.tokenSymbol})`)
      } catch (error) {
        this.resultMessage = '❌ 获取代币信息失败: ' + error.message
        this.resultType = 'error'
        this.$emit('debug-info', `获取代币信息错误: ${error.message}`)
      }
    },
    async useFaucet() {
      if (!this.stablecoinContract || !web3Service.getAccount()) {
        this.resultMessage = '❌ 请先连接钱包并设置合约'
        this.resultType = 'error'
        return
      }

      try {
        this.resultMessage = '🔄 领取测试代币中...'
        this.resultType = 'loading'

        const result = await this.stablecoinContract.methods.useFaucet().send({
          from: web3Service.getAccount(),
        })

        this.resultMessage = `✅ 成功领取测试代币！<br>交易哈希: ${result.transactionHash}`
        this.resultType = 'success'
        this.$emit('debug-info', `水龙头交易: ${result.transactionHash}`)

        setTimeout(() => this.checkBalance(), 2000)
      } catch (error) {
        this.resultMessage = '❌ 领取测试代币失败: ' + error.message
        this.resultType = 'error'
        this.$emit('debug-info', `水龙头错误: ${error.message}`)
      }
    },
    async checkBalance() {
      if (!this.stablecoinContract || !web3Service.getAccount()) {
        this.resultMessage = '❌ 请先连接钱包并设置合约'
        this.resultType = 'error'
        return
      }

      try {
        this.resultMessage = '🔄 查询余额中...'
        this.resultType = 'loading'

        const balance = await this.stablecoinContract.methods
          .balanceOf(web3Service.getAccount())
          .call()
        const symbol = await this.stablecoinContract.methods.symbol().call()

        const formattedBalance = web3Service.web3.utils.fromWei(balance, 'ether')

        this.resultMessage = `💰 您的余额: ${formattedBalance} ${symbol}`
        this.resultType = 'success'
        this.$emit('debug-info', `余额查询: ${formattedBalance} ${symbol}`)
      } catch (error) {
        this.resultMessage = '❌ 查询余额失败: ' + error.message
        this.resultType = 'error'
        this.$emit('debug-info', `余额查询错误: ${error.message}`)
      }
    },
  },
}
</script>

<style scoped></style>
