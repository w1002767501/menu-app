<template>
  <div class="menu-app">
    <!-- 顶部 -->
    <header class="header">
      <h1 class="title">今天想吃什么？</h1>
      <p class="subtitle">选好了告诉我，我来做 ♡</p>
    </header>

    <!-- 分类 -->
    <nav class="categories">
      <button
        v-for="cat in categories"
        :key="cat.id"
        :class="['cat-btn', { active: activeCategory === cat.id }]"
        @click="activeCategory = cat.id"
      >
        {{ cat.icon }} {{ cat.name }}
      </button>
    </nav>

    <!-- 菜单列表 -->
    <main class="dish-list">
      <div
        v-for="dish in filteredDishes"
        :key="dish.id"
        class="dish-card"
      >
        <div class="dish-icon">{{ dish.emoji }}</div>
        <div class="dish-body">
          <div class="dish-name">{{ dish.name }}</div>
          <div class="dish-desc">{{ dish.desc }}</div>
        </div>
        <div class="dish-action">
          <template v-if="getQty(dish.id) > 0">
            <button class="qty-btn" @click="decrease(dish)">−</button>
            <span class="qty-num">{{ getQty(dish.id) }}</span>
          </template>
          <button class="add-btn" @click="add(dish)">+</button>
        </div>
      </div>
    </main>

    <!-- 购物车按钮 -->
    <button
      v-if="totalCount > 0"
      class="cart-fab"
      @click="showCart = true"
    >
      🛒 已选 {{ totalCount }} 件
    </button>

    <!-- 购物车弹窗 -->
    <transition name="cart">
      <div
        v-if="showCart"
        class="cart-overlay"
        @click.self="showCart = false"
      >
        <div class="cart-modal">
          <div class="cart-head">
            <h2>已选菜单</h2>
            <button class="close-btn" @click="showCart = false">×</button>
          </div>
          <div class="cart-body">
            <div
              v-for="item in cart"
              :key="item.dish.id"
              class="cart-row"
            >
              <span class="cart-emoji">{{ item.dish.emoji }}</span>
              <span class="cart-name">{{ item.dish.name }}</span>
              <div class="cart-qty">
                <button @click="cartDecrease(item)">−</button>
                <span>{{ item.qty }}</span>
                <button @click="item.qty++">+</button>
              </div>
            </div>
            <div v-if="cart.length === 0" class="cart-empty">
              还没选呢~
            </div>
          </div>
          <div class="cart-foot" v-if="cart.length > 0">
            <button class="btn-clear" @click="cart = []; showCart = false">
              清空
            </button>
            <button
              :class="['btn-send', { sent: orderSent }]"
              @click="sendOrder"
            >
              {{ orderSent ? '已复制，去粘贴发送 ✓' : '📋 复制菜单发给他' }}
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'MenuApp',
  data() {
    return {
      activeCategory: 0,
      showCart: false,
      orderSent: false,
      cart: [],
      categories: [
        { id: 0, name: '全部', icon: '🍽️' },
        { id: 1, name: '主食', icon: '🍜' },
        { id: 2, name: '热菜', icon: '🥘' },
        { id: 3, name: '小食', icon: '🥗' },
        { id: 4, name: '甜品', icon: '🍰' },
        { id: 5, name: '饮品', icon: '🧋' }
      ],
      // ============================
      // 👇 在这里修改你自己的菜单！
      // ============================
      dishes: [
        { id: 1,  name: '蛋炒饭',       category: 1, emoji: '🍚', desc: '粒粒分明，蛋香四溢' },
        { id: 2,  name: '西红柿鸡蛋面', category: 1, emoji: '🍝', desc: '酸甜汤底，手擀面' },
        { id: 3,  name: '饺子',         category: 1, emoji: '🥟', desc: '猪肉白菜馅儿' },
        { id: 4,  name: '手抓饼',       category: 1, emoji: '🫓', desc: '加蛋加肠加生菜' },
        { id: 5,  name: '红烧肉',       category: 2, emoji: '🥩', desc: '肥而不腻，入口即化' },
        { id: 6,  name: '可乐鸡翅',     category: 2, emoji: '🍗', desc: '甜香入味，吮指回味' },
        { id: 7,  name: '番茄炒蛋',     category: 2, emoji: '🍳', desc: '永远的国民家常菜' },
        { id: 8,  name: '麻婆豆腐',     category: 2, emoji: '🌶️', desc: '麻辣鲜香，下饭神器' },
        { id: 9,  name: '酸菜鱼',       category: 2, emoji: '🐟', desc: '酸辣开胃，鱼片嫩滑' },
        { id: 10, name: '蒜蓉西兰花',   category: 2, emoji: '🥦', desc: '清淡健康' },
        { id: 11, name: '拍黄瓜',       category: 3, emoji: '🥒', desc: '蒜香清爽' },
        { id: 12, name: '炸鸡块',       category: 3, emoji: '🍟', desc: '外酥里嫩' },
        { id: 13, name: '双皮奶',       category: 4, emoji: '🍮', desc: '滑嫩香甜' },
        { id: 14, name: '芒果千层',     category: 4, emoji: '🥭', desc: '层层叠叠的甜蜜' },
        { id: 15, name: '草莓蛋糕',     category: 4, emoji: '🍓', desc: '新鲜草莓奶油' },
        { id: 16, name: '杨枝甘露',     category: 5, emoji: '🥤', desc: '芒果椰汁西米露' },
        { id: 17, name: '柠檬茶',       category: 5, emoji: '🍋', desc: '手打鲜柠檬' },
        { id: 18, name: '热可可',       category: 5, emoji: '☕', desc: '冬日暖心' }
      ]
    }
  },
  computed: {
    filteredDishes() {
      if (this.activeCategory === 0) return this.dishes
      return this.dishes.filter(d => d.category === this.activeCategory)
    },
    totalCount() {
      return this.cart.reduce((sum, item) => sum + item.qty, 0)
    },
    orderText() {
      const lines = ['📋 今日菜单已点好~', '']
      this.cart.forEach(item => {
        lines.push(`${item.dish.emoji} ${item.dish.name} ×${item.qty}`)
      })
      lines.push('', '💕 就这些啦，等你投喂~')
      return lines.join('\n')
    }
  },
  methods: {
    add(dish) {
      const found = this.cart.find(i => i.dish.id === dish.id)
      if (found) {
        found.qty++
      } else {
        this.cart.push({ dish, qty: 1 })
      }
    },
    decrease(dish) {
      const idx = this.cart.findIndex(i => i.dish.id === dish.id)
      if (idx === -1) return
      if (this.cart[idx].qty > 1) {
        this.cart[idx].qty--
      } else {
        this.cart.splice(idx, 1)
      }
    },
    getQty(id) {
      const item = this.cart.find(i => i.dish.id === id)
      return item ? item.qty : 0
    },
    cartDecrease(item) {
      if (item.qty > 1) {
        item.qty--
      } else {
        this.cart = this.cart.filter(i => i.dish.id !== item.dish.id)
      }
    },
    async sendOrder() {
      try {
        if (navigator.clipboard && navigator.clipboard.writeText) {
          await navigator.clipboard.writeText(this.orderText)
        } else {
          this.fallbackCopy()
        }
      } catch {
        this.fallbackCopy()
      }
      this.orderSent = true
      setTimeout(() => { this.orderSent = false }, 2500)
    },
    fallbackCopy() {
      const ta = document.createElement('textarea')
      ta.value = this.orderText
      ta.style.cssText = 'position:fixed;opacity:0'
      document.body.appendChild(ta)
      ta.focus()
      ta.select()
      document.execCommand('copy')
      document.body.removeChild(ta)
    }
  }
}
</script>

<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.menu-app {
  min-height: 100vh;
  background: #FEF7ED;
  padding-bottom: 90px;
  font-family: 'LXGW WenKai', 'PingFang SC', 'Hiragino Sans GB', sans-serif;
  max-width: 480px;
  margin: 0 auto;
  position: relative;
}

/* ===== Header ===== */
.header {
  padding: 44px 24px 28px;
  background: linear-gradient(135deg, #E85D4A 0%, #F2845C 100%);
  color: #fff;
  border-radius: 0 0 28px 28px;
  position: relative;
  overflow: hidden;
}
.header::before {
  content: '';
  position: absolute;
  width: 200px; height: 200px;
  background: rgba(255,255,255,0.07);
  border-radius: 50%;
  top: -60px; right: -40px;
}
.header::after {
  content: '';
  position: absolute;
  width: 120px; height: 120px;
  background: rgba(255,255,255,0.05);
  border-radius: 50%;
  bottom: -30px; left: -20px;
}
.title {
  font-size: 26px;
  font-weight: 700;
  position: relative; z-index: 1;
}
.subtitle {
  font-size: 14px;
  margin-top: 6px;
  opacity: 0.9;
  position: relative; z-index: 1;
}

/* ===== Categories ===== */
.categories {
  display: flex;
  gap: 8px;
  padding: 18px 20px 10px;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none;
}
.categories::-webkit-scrollbar { display: none; }
.cat-btn {
  flex-shrink: 0;
  padding: 7px 14px;
  border: none;
  border-radius: 20px;
  background: #fff;
  color: #2D1F14;
  font-size: 13px;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 1px 3px rgba(45,31,20,0.06);
  white-space: nowrap;
}
.cat-btn.active {
  background: #E85D4A;
  color: #fff;
  box-shadow: 0 2px 10px rgba(232,93,74,0.3);
}
.cat-btn:active { transform: scale(0.95); }

/* ===== Dish List ===== */
.dish-list { padding: 10px 16px 0; }
.dish-card {
  display: flex;
  align-items: center;
  background: #fff;
  border-radius: 16px;
  padding: 14px 16px;
  margin-bottom: 10px;
  box-shadow: 0 1px 4px rgba(45,31,20,0.05);
  animation: cardIn 0.35s ease;
}
@keyframes cardIn {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
.dish-icon {
  font-size: 32px;
  width: 52px; height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #FEF7ED;
  border-radius: 14px;
  flex-shrink: 0;
}
.dish-body { flex: 1; margin: 0 12px; min-width: 0; }
.dish-name { font-size: 15px; font-weight: 600; color: #2D1F14; }
.dish-desc {
  font-size: 12px;
  color: #9C8B7E;
  margin-top: 3px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.dish-action { display: flex; align-items: center; gap: 6px; flex-shrink: 0; }
.add-btn, .qty-btn {
  width: 30px; height: 30px;
  border-radius: 50%;
  border: none;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  font-family: inherit;
}
.add-btn { background: #E85D4A; color: #fff; }
.add-btn:active { transform: scale(0.85); }
.qty-btn {
  background: transparent;
  border: 1.5px solid #E85D4A;
  color: #E85D4A;
  font-size: 16px;
}
.qty-btn:active { background: #FEEEE9; }
.qty-num {
  font-size: 15px;
  font-weight: 700;
  min-width: 20px;
  text-align: center;
  color: #2D1F14;
}

/* ===== Cart FAB ===== */
.cart-fab {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #E85D4A;
  color: #fff;
  border: none;
  border-radius: 28px;
  padding: 12px 28px;
  font-size: 15px;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(232,93,74,0.4);
  z-index: 100;
  transition: transform 0.2s ease;
  animation: fabIn 0.3s cubic-bezier(0.34,1.56,0.64,1);
}
@keyframes fabIn {
  from { transform: translateX(-50%) scale(0) translateY(20px); opacity: 0; }
  to   { transform: translateX(-50%) scale(1) translateY(0); opacity: 1; }
}
.cart-fab:active { transform: translateX(-50%) scale(0.95); }

/* ===== Cart Modal ===== */
.cart-enter-active { transition: opacity 0.3s ease; }
.cart-enter-active .cart-modal { transition: transform 0.35s cubic-bezier(0.16,1,0.3,1); }
.cart-leave-active { transition: opacity 0.25s ease; }
.cart-leave-active .cart-modal { transition: transform 0.25s ease; }
.cart-enter, .cart-leave-to { opacity: 0; }
.cart-enter .cart-modal, .cart-leave-to .cart-modal { transform: translateY(100%); }

.cart-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.35);
  z-index: 200;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}
.cart-modal {
  background: #fff;
  width: 100%;
  max-width: 480px;
  border-radius: 24px 24px 0 0;
  max-height: 65vh;
  display: flex;
  flex-direction: column;
}
.cart-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px 14px;
}
.cart-head h2 { font-size: 18px; color: #2D1F14; }
.close-btn {
  width: 30px; height: 30px;
  border-radius: 50%;
  border: none;
  background: #f5f0e8;
  color: #9C8B7E;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cart-body { flex: 1; overflow-y: auto; padding: 0 24px 16px; }
.cart-row {
  display: flex;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f5f0e8;
}
.cart-emoji { font-size: 22px; margin-right: 10px; }
.cart-name { flex: 1; font-size: 14px; color: #2D1F14; }
.cart-qty { display: flex; align-items: center; gap: 8px; }
.cart-qty button {
  width: 26px; height: 26px;
  border-radius: 50%;
  border: 1.5px solid #e0d8cf;
  background: transparent;
  color: #2D1F14;
  font-size: 15px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: inherit;
}
.cart-qty button:active { background: #f5f0e8; }
.cart-qty span { font-weight: 600; min-width: 18px; text-align: center; font-size: 14px; }
.cart-empty { text-align: center; padding: 32px 0; color: #9C8B7E; font-size: 14px; }

.cart-foot {
  display: flex;
  gap: 10px;
  padding: 14px 24px 28px;
  border-top: 1px solid #f0ebe4;
}
.btn-clear {
  flex: 1;
  padding: 12px;
  border: 1.5px solid #e0d8cf;
  border-radius: 12px;
  background: transparent;
  color: #9C8B7E;
  font-size: 14px;
  font-family: inherit;
  cursor: pointer;
}
.btn-clear:active { background: #f5f0e8; }
.btn-send {
  flex: 2;
  padding: 12px;
  border: none;
  border-radius: 12px;
  background: #E85D4A;
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  transition: background 0.3s ease;
}
.btn-send:active { opacity: 0.9; }
.btn-send.sent { background: #4A9B6E; }
</style>