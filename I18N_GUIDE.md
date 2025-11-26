# 国际化（i18n）集成指南

## 概述

项目已完全集成英文和中文多语言支持。用户可通过右上角的语言切换按钮在 English（英文）和 中文 之间切换。

## 文件说明

### 1. 翻译文件 `src/utils/i18n.js`

包含两个语言的完整翻译：

```javascript
export const translations = {
  en: {
    /* 英文翻译 */
  },
  zh: {
    /* 中文翻译 */
  },
}

export function getTranslation(lang, key) {
  // 根据语言和路径返回翻译文本
}
```

**翻译结构：**

- `walletConnection.*` - 钱包连接相关文本
- `contractSetup.*` - 合约设置相关文本
- `tabs.*` - 标签页标题
- `stablecoinTest.*` - 稳定币测试文本
- `nftTest.*` - NFT测试文本
- `marketplaceDirect.*` - 直接购买市场文本
- `marketplaceAuction.*` - 拍卖市场文本
- `debugInfo.*` - 调试信息文本
- `common.*` - 通用文本（加载、成功、错误等）

### 2. 语言切换组件 `src/components/LanguageSwitcher.vue`

- 位置：页面右上角
- 功能：显示 English 和 中文 两个按钮
- 选中状态：活跃按钮显示蓝色背景
- 持久化：所选语言保存到 localStorage

### 3. App.vue 更新

```vue
<script>
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import { getTranslation } from './utils/i18n.js'

export default {
  data() {
    return {
      language: 'en', // 默认英文
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
  },
}
</script>
```

## 所有更新的组件

| 组件名                   | 更新内容                                     |
| ------------------------ | -------------------------------------------- |
| `WalletConnection.vue`   | 添加 language prop，所有文本使用 t() 方法    |
| `ContractSetup.vue`      | 添加 language prop，支持三个合约的多语言设置 |
| `StablecoinTest.vue`     | 添加 language prop，所有操作文本国际化       |
| `NFTTest.vue`            | 添加 language prop，铸造、查询等操作国际化   |
| `MarketplaceDirect.vue`  | 添加 language prop，购买和取消操作国际化     |
| `MarketplaceAuction.vue` | 添加 language prop，竞拍操作国际化           |
| `DebugInfo.vue`          | 添加 language prop，调试界面国际化           |
| `LanguageSwitcher.vue`   | **新组件**，语言切换功能                     |

## 使用方式

### 对于用户

1. 点击右上角的语言按钮（English / 中文）切换语言
2. 所有界面文本和消息会实时更新
3. 语言选择会自动保存，刷新页面后保持选择

### 对于开发者

#### 在 App.vue 中使用

```vue
<template>
  <div class="container">
    <LanguageSwitcher v-model="language" />
    <h1>{{ t('title') }}</h1>
    <!-- 传递 language 给所有子组件 -->
    <WalletConnection :language="language" />
  </div>
</template>

<script>
import { getTranslation } from './utils/i18n'

export default {
  data() {
    return {
      language: 'en',
    }
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
  },
}
</script>
```

#### 在子组件中使用

```vue
<script>
import { getTranslation } from '../utils/i18n'

export default {
  props: {
    language: {
      type: String,
      default: 'en',
    },
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
  },
}
</script>

<template>
  <h2>{{ t('walletConnection.title') }}</h2>
  <button>{{ t('walletConnection.button') }}</button>
</template>
```

## 翻译键值结构

### 示例路径

- `t('title')` → "NFT Trading Platform"（英文）/ "NFT 交易平台"（中文）
- `t('walletConnection.button')` → "Connect MetaMask" / "连接 MetaMask"
- `t('common.success')` → "✅ " （两种语言相同）

### 添加新翻译

1. 编辑 `src/utils/i18n.js`
2. 在相应的语言对象中添加新键值对

```javascript
export const translations = {
  en: {
    myNewFeature: {
      title: 'My Feature',
      button: 'Click Me',
    },
  },
  zh: {
    myNewFeature: {
      title: '我的功能',
      button: '点击我',
    },
  },
}
```

3. 在组件中使用

```javascript
{
  {
    t('myNewFeature.title')
  }
}
{
  {
    t('myNewFeature.button')
  }
}
```

## 调试技巧

### 检查翻译加载

在浏览器控制台运行：

```javascript
import { translations } from './src/utils/i18n.js'
console.log(translations.en) // 查看英文翻译
console.log(translations.zh) // 查看中文翻译
```

### 切换语言进行测试

```javascript
localStorage.setItem('language', 'zh') // 切换到中文
localStorage.setItem('language', 'en') // 切换到英文
// 刷新页面查看效果
```

## 已翻译的内容清单

### 英文版本 (en)

- ✅ 页面标题和描述
- ✅ 所有按钮文本
- ✅ 成功/错误消息
- ✅ 所有表单标签
- ✅ 合约操作提示
- ✅ 调试信息

### 中文版本 (zh)

- ✅ 所有对应的中文翻译
- ✅ 保留原有的 emoji 图标
- ✅ 自然的中文表达

## 常见问题

**Q: 如何修改默认语言？**
A: 在 App.vue 的 mounted() 中修改，或检查 localStorage 中的 'language' 值。

**Q: 如何添加第三种语言？**
A:

1. 在 `i18n.js` 中添加新语言对象
2. 在 `LanguageSwitcher.vue` 的 languages 数组中添加新选项
3. 在所有组件中使用相同的 getTranslation 方法

**Q: 翻译没有更新？**
A: 检查：

1. 组件是否接收了 `language` prop
2. 是否使用了 `t()` 方法
3. localStorage 中的 'language' 值

## 部署注意事项

- 不需要额外的 npm 包
- 所有翻译都在 `i18n.js` 文件中
- localStorage 用于持久化语言选择
- 完全客户端实现，无后端依赖

---

**维护者注意**：如需添加新翻译或修改现有翻译，请编辑 `src/utils/i18n.js` 文件，然后测试所有组件确保翻译正确显示。
