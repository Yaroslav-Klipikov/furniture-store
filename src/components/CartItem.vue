<script setup>
defineProps({
  item: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['increment', 'decrement', 'remove'])

const formatPrice = (price) => {
  return price.toLocaleString('uk-UA') + ' ₴'
}

const handleImageError = (event) => {
  event.target.src = 'https://placehold.co/150x150/ece7e1/5c4d3c?text=DOMUS'
}
</script>

<template>
  <div class="cart-item">
    <img
      :src="item.image"
      :alt="item.name"
      class="cart-item-img"
      @error="handleImageError"
    />

    <div class="cart-item-info">
      <div class="cart-item-top">
        <h4 class="cart-item-title">{{ item.name }}</h4>
        <button
          type="button"
          class="btn-remove"
          title="Видалити позицію"
          aria-label="Видалити позицію"
          @click="emit('remove', item.id)"
        >
          ✕
        </button>
      </div>

      <p class="cart-item-unit">{{ formatPrice(item.price) }} / шт.</p>

      <div class="cart-item-bottom">
        <div class="qty-control">
          <button
            type="button"
            class="qty-btn"
            aria-label="Зменшити кількість"
            @click="emit('decrement', item.id)"
          >
            −
          </button>
          <span class="qty-value">{{ item.quantity }}</span>
          <button
            type="button"
            class="qty-btn"
            aria-label="Збільшити кількість"
            @click="emit('increment', item.id)"
          >
            +
          </button>
        </div>

        <strong class="cart-item-subtotal">
          {{ formatPrice(item.price * item.quantity) }}
        </strong>
      </div>
    </div>
  </div>
</template>