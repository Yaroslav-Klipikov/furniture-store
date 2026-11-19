<script setup>

import { ref, computed, watch, onMounted } from 'vue'
import ProductCard from './components/ProductCard.vue'
import CartItem from './components/CartItem.vue'



const STORAGE_KEY = 'domus_furniture_cart'

// Завантаження збереженого кошика з localStorage
const loadSavedCart = () => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    return raw ? JSON.parse(raw) : []
  } catch {
    return []
  }
}

// Стан каталогу
const products = ref([])
const isLoading = ref(true)
const loadError = ref('')

// Фільтри та пошук
const selectedCategory = ref('Усі')
const searchQuery = ref('')
const sortBy = ref('default')

// Стан кошика
const cart = ref(loadSavedCart())
const isCartOpen = ref(false)

// Промокод
const promoInput = ref('')
const appliedPromo = ref('')
const promoFeedback = ref('')

// Форма оформлення замовлення
const showCheckout = ref(false)
const orderSuccess = ref(false)
const checkoutForm = ref({
  name: '',
  phone: '',
  address: '',
  paymentMethod: 'apple-pay'
})

const formErrors = ref({
  name: '',
  phone: '',
  address: ''
})

// Завантаження JSON через fetch
onMounted(async () => {
  try {
    const response = await fetch('/products.json')
    if (!response.ok) {
      throw new Error('Не вдалося завантажити каталог товарів.')
    }
    products.value = await response.json()
  } catch (err) {
    loadError.value = err.message || 'Помилка завантаження даних.'
  } finally {
    isLoading.value = false
  }
})

// Автоматичне збереження кошика в localStorage при кожній зміні
watch(
  cart,
  (newCart) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newCart))
  },
  { deep: true }
)

// Список категорій для фільтра
const categories = computed(() => {
  const unique = new Set(products.value.map((p) => p.category))
  return ['Усі', ...unique]
})

// Фільтрація за категорією, пошук за назвою та сортування (computed)
const filteredProducts = computed(() => {
  const query = searchQuery.value.trim().toLowerCase()

  let result = products.value.filter((product) => {
    const matchesCategory =
      selectedCategory.value === 'Усі' ||
      product.category === selectedCategory.value
    const matchesSearch =
      !query ||
      product.name.toLowerCase().includes(query) ||
      product.description.toLowerCase().includes(query)
    return matchesCategory && matchesSearch
  })

  if (sortBy.value === 'price-asc') {
    result = [...result].sort((a, b) => a.price - b.price)
  } else if (sortBy.value === 'price-desc') {
    result = [...result].sort((a, b) => b.price - a.price)
  }

  return result
})

// Лічильник товарів у шапці
const cartCount = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.quantity, 0)
})

// Сума без знижки
const subtotalPrice = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
})

// Знижка за промокодом HOME10 (10%)
const discountAmount = computed(() => {
  return appliedPromo.value === 'HOME10'
    ? Math.round(subtotalPrice.value * 0.1)
    : 0
})

// Фінальна сума
const totalPrice = computed(() => {
  return subtotalPrice.value - discountAmount.value
})

// Методи роботи з кошиком
const getQuantityInCart = (productId) => {
  const found = cart.value.find((item) => item.id === productId)
  return found ? found.quantity : 0
}

const addToCart = (product) => {
  orderSuccess.value = false
  const existing = cart.value.find((item) => item.id === product.id)
  if (existing) {
    existing.quantity += 1
  } else {
    cart.value.push({
      id: product.id,
      name: product.name,
      price: product.price,
      image: product.image,
      quantity: 1
    })
  }
}

const incrementItem = (id) => {
  const item = cart.value.find((i) => i.id === id)
  if (item) item.quantity += 1
}

const decrementItem = (id) => {
  const index = cart.value.findIndex((i) => i.id === id)
  if (index === -1) return
  if (cart.value[index].quantity > 1) {
    cart.value[index].quantity -= 1
  } else {
    cart.value.splice(index, 1)
  }
}

const removeItem = (id) => {
  cart.value = cart.value.filter((i) => i.id !== id)
}

const resetFilters = () => {
  selectedCategory.value = 'Усі'
  searchQuery.value = ''
  sortBy.value = 'default'
}

// Застосування промокоду
const applyPromoCode = () => {
  const code = promoInput.value.trim().toUpperCase()
  if (code === 'HOME10') {
    appliedPromo.value = 'HOME10'
    promoFeedback.value = 'Промокод HOME10 застосовано (−10%).'
  } else {
    appliedPromo.value = ''
    promoFeedback.value = 'Невірний промокод. Спробуйте HOME10.'
  }
}

// Валідація та відправка форми замовлення
const submitOrder = () => {
  formErrors.value = { name: '', phone: '', address: '' }
  let isValid = true

  if (checkoutForm.value.name.trim().length < 2) {
    formErrors.value.name = 'Введіть ім’я (мінімум 2 символи).'
    isValid = false
  }

  const phoneClean = checkoutForm.value.phone.replace(/[\s\-()]/g, '')
  if (!/^\+?\d{10,13}$/.test(phoneClean)) {
    formErrors.value.phone = 'Введіть коректний номер телефону (наприклад, +380991234567).'
    isValid = false
  }

  if (checkoutForm.value.address.trim().length < 5) {
    formErrors.value.address = 'Вкажіть адресу доставки.'
    isValid = false
  }

  if (!isValid) return

  cart.value = []
  appliedPromo.value = ''
  promoInput.value = ''
  promoFeedback.value = ''
  showCheckout.value = false
  orderSuccess.value = true
  checkoutForm.value = { name: '', phone: '', address: '' }
}

const formatPrice = (price) => {
  return price.toLocaleString('uk-UA') + ' ₴'
}
</script>

<template>
  <div class="app-wrapper">
    <!-- Шапка сайту -->
    <header class="header">
      <div class="container header-inner">
        <div class="brand">
          <span class="brand-logo">DOMUS</span>
          <span class="brand-sub">Меблі та гарнітура для дому</span>
        </div>

        <button
          type="button"
          class="cart-toggle-btn"
          @click="isCartOpen = true"
        >
          <span>Кошик</span>
          <span class="cart-badge">{{ cartCount }}</span>
          <span v-if="totalPrice > 0" class="cart-header-sum">
            {{ formatPrice(totalPrice) }}
          </span>
        </button>
      </div>
    </header>

    <!-- Основний контент -->
    <main class="container main-content">
      <section class="hero">
        <h1>Сучасні меблі для квартири та будинку</h1>
        <p>
          Лаконічний дизайн, натуральні матеріали та швидка доставка. Використовуйте промокод
          <strong>HOME10</strong> для знижки 10%.
        </p>
      </section>

      <!-- Панель пошуку, фільтрації та сортування -->
      <section class="controls-bar" aria-label="Фільтрація каталогу">
        <div class="search-wrap">
          <input
            v-model="searchQuery"
            type="search"
            class="search-input"
            placeholder="Пошук меблів за назвою..."
            aria-label="Пошук за назвою"
          />
        </div>

        <div class="sort-wrap">
          <select v-model="sortBy" class="sort-select" aria-label="Сортування">
            <option value="default">За замовчуванням</option>
            <option value="price-asc">Спочатку дешевші</option>
            <option value="price-desc">Спочатку дорожчі</option>
          </select>
        </div>
      </section>

      <!-- Кнопки категорій -->
      <nav class="categories-nav" aria-label="Категорії товарів">
        <button
          v-for="category in categories"
          :key="category"
          type="button"
          class="category-btn"
          :class="{ active: selectedCategory === category }"
          @click="selectedCategory = category"
        >
          {{ category }}
        </button>
      </nav>

      <!-- Стани завантаження / помилки / порожнього результату -->
      <div v-if="isLoading" class="state-box">
        Завантаження каталогу...
      </div>

      <div v-else-if="loadError" class="state-box state-error">
        {{ loadError }}
      </div>

      <div v-else-if="filteredProducts.length === 0" class="state-box">
        <h3>Нічого не знайдено</h3>
        <p>За вашим запитом товарів не знайдено. Спробуйте змінити категорію або текст пошуку.</p>
        <button type="button" class="btn-secondary" @click="resetFilters">
          Скинути фільтри
        </button>
      </div>

      <!-- Сітка товарів -->
    <section v-else class="catalog-grid">
        <ProductCard
          v-for="product in filteredProducts"
          :key="product.id"
          :product="product"
          :quantity-in-cart="getQuantityInCart(product.id)"
          @add-to-cart="addToCart"
          @decrement="decrementItem"
        />
      </section>
    </main>


    <!-- Футер -->
    <footer class="footer">
      <div class="container">
        <p>© 2026 DOMUS — Магазин меблів та гарнітури. КП № 2 на Vue.js 3.</p>
      </div>
    </footer>

    <!-- Висувна панель кошика -->
    <div
      v-if="isCartOpen"
      class="cart-overlay"
      @click.self="isCartOpen = false"
    >
      <aside class="cart-drawer" aria-label="Кошик покупок">
        <div class="cart-header">
          <h2>Кошик ({{ cartCount }})</h2>
          <button
            type="button"
            class="btn-close"
            aria-label="Закрити кошик"
            @click="isCartOpen = false"
          >
            ✕
          </button>
        </div>

        <div class="cart-body">
          <!-- Повідомлення про успішне замовлення -->
          <div v-if="orderSuccess" class="state-box state-success">
            <h3>Замовлення оформлено!</h3>
            <p>Дякуємо за покупку. Наш менеджер зв’яжеться з вами найближчим часом.</p>
            <button
              type="button"
              class="btn-primary"
              @click="isCartOpen = false; orderSuccess = false"
            >
              Повернутися до каталогу
            </button>
          </div>

          <!-- Повідомлення для порожнього кошика -->
          <div v-else-if="cart.length === 0" class="state-box">
            <h3>Кошик порожній</h3>
            <p>Додайте меблі або гарнітуру з каталогу, щоб оформити замовлення.</p>
          </div>

          <!-- Список товарів у кошику -->
          <template v-else>
            <div class="cart-items-list">
              <CartItem
                v-for="item in cart"
                :key="item.id"
                :item="item"
                @increment="incrementItem"
                @decrement="decrementItem"
                @remove="removeItem"
              />
            </div>

            <!-- Блок промокоду -->
            <div class="promo-box">
              <input
                v-model="promoInput"
                type="text"
                class="promo-input"
                placeholder="Промокод (HOME10)"
              />
              <button
                type="button"
                class="btn-secondary"
                @click="applyPromoCode"
              >
                Застосувати
              </button>
            </div>
            <p
              v-if="promoFeedback"
              class="promo-feedback"
              :class="{ valid: appliedPromo === 'HOME10' }"
            >
              {{ promoFeedback }}
            </p>

            <!-- Підсумки -->
            <div class="cart-summary">
              <div class="summary-row">
                <span>Товари ({{ cartCount }} шт.):</span>
                <span>{{ formatPrice(subtotalPrice) }}</span>
              </div>
              <div v-if="discountAmount > 0" class="summary-row discount">
                <span>Знижка (HOME10):</span>
                <span>−{{ formatPrice(discountAmount) }}</span>
              </div>
              <div class="summary-row total">
                <span>До сплати:</span>
                <span>{{ formatPrice(totalPrice) }}</span>
              </div>
            </div>

            <!-- Кнопка переходу до форми або сама форма -->
            <button
              v-if="!showCheckout"
              type="button"
              class="btn-primary btn-full"
              @click="showCheckout = true"
            >
              Оформити замовлення
            </button>

            <form
              v-else
              class="checkout-form"
              novalidate
              @submit.prevent="submitOrder"
            >
              <h3>Дані для доставки</h3>

              <div class="form-group">
                <label>Спосіб оплати</label>
                <div class="payment-methods">
                  <button
                    type="button"
                    class="payment-option"
                    :class="{ active: checkoutForm.paymentMethod === 'apple-pay' }"
                    @click="checkoutForm.paymentMethod = 'apple-pay'"
                  >
                     Apple Pay
                  </button>
                  <button
                    type="button"
                    class="payment-option"
                    :class="{ active: checkoutForm.paymentMethod === 'google-pay' }"
                    @click="checkoutForm.paymentMethod = 'google-pay'"
                  >
                    G Pay
                  </button>
                  <button
                    type="button"
                    class="payment-option"
                    :class="{ active: checkoutForm.paymentMethod === 'visa' }"
                    @click="checkoutForm.paymentMethod = 'visa'"
                  >
                    VISA
                  </button>
                  <button
                    type="button"
                    class="payment-option"
                    :class="{ active: checkoutForm.paymentMethod === 'mastercard' }"
                    @click="checkoutForm.paymentMethod = 'mastercard'"
                  >
                    Mastercard
                  </button>
                </div>
              </div>
              
              <div class="form-group">
                <label for="customer-name">Ім’я та прізвище</label>
                <input
                  id="customer-name"
                  v-model="checkoutForm.name"
                  type="text"
                  placeholder="Олександр Коваленко"
                />
                <span v-if="formErrors.name" class="field-error">{{ formErrors.name }}</span>
              </div>

              <div class="form-group">
                <label for="customer-phone">Телефон</label>
                <input
                  id="customer-phone"
                  v-model="checkoutForm.phone"
                  type="tel"
                  placeholder="+380991234567"
                />
                <span v-if="formErrors.phone" class="field-error">{{ formErrors.phone }}</span>
              </div>

              <div class="form-group">
                <label for="customer-address">Адреса доставки</label>
                <input
                  id="customer-address"
                  v-model="checkoutForm.address"
                  type="text"
                  placeholder="м. Київ, вул. Хрещатик, 10"
                />
                <span v-if="formErrors.address" class="field-error">{{ formErrors.address }}</span>
              </div>

              <div class="checkout-actions">
                <button type="submit" class="btn-primary btn-full">
                  Підтвердити замовлення
                </button>
                <button
                  type="button"
                  class="btn-secondary btn-full"
                  @click="showCheckout = false"
                >
                  Скасувати
                </button>
              </div>
            </form>
          </template>
        </div>
      </aside>
    </div>
  </div>
</template>