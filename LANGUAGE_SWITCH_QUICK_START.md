# 多语言功能快速开始

## 🎉 新增功能

项目已支持完整的英文和中文国际化！

## 🚀 快速使用

### 启动项目

```bash
npm install
npm run dev
```

打开 `http://localhost:5173`

### 切换语言

**右上角找到语言切换按钮：**

```
┌──────────────────────────────┐
│ [English] [中文]             │  ← 右上角
└──────────────────────────────┘
```

- 点击 **English** → 页面全部转为英文
- 点击 **中文** → 页面全部转为中文
- 你的选择会自动保存，刷新页面后仍保持选择

## 🌍 支持的语言

| 语言     | 按钮    | 代码 |
| -------- | ------- | ---- |
| English  | English | `en` |
| 简体中文 | 中文    | `zh` |

## 📱 页面元素的翻译

### 所有页面元素已翻译：

#### 1. 钱包连接

- ✅ "Connect Wallet" / "连接钱包"
- ✅ 所有连接状态消息
- ✅ 钱包地址、网络、链ID标签

#### 2. 合约设置

- ✅ "Stablecoin Contract" / "稳定币合约"
- ✅ "NFT Contract" / "NFT合约"
- ✅ "Marketplace Contract" / "市场合约"
- ✅ 所有验证消息

#### 3. 标签页标题

- ✅ Stablecoin Test / 稳定币测试
- ✅ NFT Test / NFT测试
- ✅ Marketplace Direct Purchase / 市场直接购买
- ✅ Marketplace Auction / 市场拍卖
- ✅ Debug Info / 调试信息

#### 4. 所有操作按钮和提示消息

## 🔧 技术实现

### 核心文件

```
src/
├── utils/
│   └── i18n.js           ← 翻译文件
├── components/
│   ├── LanguageSwitcher.vue  ← 语言切换器
│   ├── App.vue           ← 主应用（已更新）
│   ├── WalletConnection.vue  ← 已支持i18n
│   ├── ContractSetup.vue     ← 已支持i18n
│   ├── StablecoinTest.vue    ← 已支持i18n
│   ├── NFTTest.vue           ← 已支持i18n
│   ├── MarketplaceDirect.vue ← 已支持i18n
│   ├── MarketplaceAuction.vue ← 已支持i18n
│   └── DebugInfo.vue         ← 已支持i18n
```

### 工作流程

```
用户点击语言按钮
    ↓
LanguageSwitcher 更新 App.language
    ↓
App 传递 :language 给所有子组件
    ↓
子组件调用 t(key) 方法
    ↓
getTranslation(language, key) 返回对应文本
    ↓
界面更新显示选中语言的文本
```

## 💾 数据持久化

所选语言自动保存到浏览器 localStorage：

```javascript
// 语言选择保存在
localStorage.getItem('language') // 返回 'en' 或 'zh'
```

## 🎨 界面效果

### 语言切换器样式

```
┌─────────┬─────────┐
│English ▌│ 中文   │  ← 英文选中时
└─────────┴─────────┘

┌─────────┬─────────┐
│ English │▌中文   │  ← 中文选中时
└─────────┴─────────┘
```

- 选中的语言显示蓝色背景
- 鼠标悬停显示深色背景
- 固定在右上角，不遮挡内容

## 📋 翻译内容示例

### 英文版本

```
Connect Wallet
Stablecoin Contract
Get Token Info
Claim Test Tokens
Check Balance
✅ Token info retrieved successfully!
```

### 中文版本

```
连接钱包
稳定币合约
🔍 获取代币信息
🚰 领取测试代币
💰 查询余额
✅ 代币信息获取成功！
```

## 🔍 调试

### 检查当前语言

打开浏览器开发者工具 (F12)，在 Console 中运行：

```javascript
localStorage.getItem('language')
```

### 手动切换语言

```javascript
// 切换到中文
localStorage.setItem('language', 'zh')
location.reload()

// 切换到英文
localStorage.setItem('language', 'en')
location.reload()
```

### 查看所有翻译

```javascript
import { translations } from './src/utils/i18n.js'
console.log(translations)
```

## 🎯 最常用的功能

### 连接钱包（英文流程）

1. 启动应用
2. 点击右上角 **English**（确保英文）
3. 点击 **Connect MetaMask**
4. 在 MetaMask 中确认连接
5. 看到 ✅ "MetaMask connected successfully!"

### 连接钱包（中文流程）

1. 启动应用
2. 点击右上角 **中文**
3. 点击 **连接 MetaMask**
4. 在 MetaMask 中确认连接
5. 看到 ✅ "MetaMask 连接成功！"

## ⚙️ 高级用法

### 添加新语言

如需添加第三种语言（如日语），请编辑 `src/utils/i18n.js`：

```javascript
export const translations = {
  en: {
    /* 英文 */
  },
  zh: {
    /* 中文 */
  },
  ja: {
    /* 日文 */
  }, // 新增
}
```

然后更新 `LanguageSwitcher.vue`：

```javascript
languages: [
  { code: 'en', name: 'English' },
  { code: 'zh', name: '中文' },
  { code: 'ja', name: '日本語' }, // 新增
]
```

### 修改翻译

编辑 `src/utils/i18n.js` 中的相应文本：

```javascript
walletConnection: {
  button: 'Connect MetaMask',  // 修改这里
}
```

修改后自动生效，无需重启。

## 📊 翻译覆盖率

- ✅ 页面标题：100%
- ✅ 按钮文本：100%
- ✅ 提示消息：100%
- ✅ 错误消息：100%
- ✅ 标签页：100%
- ✅ 表单标签：100%

## 🐛 故障排除

### 问题：切换语言后文本没有更新

**解决方案：**

1. 检查浏览器缓存（Ctrl+Shift+Delete）
2. 确认组件接收了 `language` prop
3. 检查控制台是否有错误

### 问题：某些文本仍然是中文/英文

**检查：**

1. 该文本是否在翻译文件中定义
2. 组件是否使用了 `{{ t('key') }}`
3. 是否清除了浏览器缓存

### 问题：刷新后语言重置为英文

**原因：**

- localStorage 被清除或浏览器不支持
- 可能在隐私浏览模式下

**解决方案：**

- 使用非隐私窗口
- 检查浏览器 localStorage 权限

## 📚 更多文档

- **完整i18n指南**: 查看 `I18N_GUIDE.md`
- **项目结构**: 查看 `PROJECT_STRUCTURE.md`
- **快速开始**: 查看 `QUICK_START.md`

---

**现在就试试吧！** 🌟

启动项目，点击右上角的语言按钮，体验完整的中英文切换！
