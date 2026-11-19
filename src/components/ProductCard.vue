<script setup>
defineProps({
  product: {
    type: Object,
    required: true
  },
  quantityInCart: {
    type: Number,
    default: 0
  }
})

const emit = defineEmits(['add-to-cart'])

const formatPrice = (price) => {
  return price.toLocaleString('uk-UA') + ' ₴'
}

const handleImageError = (event) => {
  event.target.src = 'https://placehold.co/600x400/ece7e1/5c4d3c?text=DOMUS+Меблі'
}
</script>

<template>
  <article class="product-card">
    <div class="product-image-wrap">
      <img
        :src="product.image"
        :alt="product.name"
        class="product-image"
        loading="lazy"
        @error="handleImageError"
      />
      <span class="product-badge">{{ product.category }}</span>
    </div>

    <div class="product-body">
      <h3 class="product-title">{{ product.name }}</h3>
      <p class="product-desc">{{ product.description }}</p>

      <div class="product-footer">
        <span class="product-price">{{ formatPrice(product.price) }}</span>
        <button
          type="button"
          class="btn-primary"
          :class="{ 'btn-in-cart': quantityInCart > 0 }"
          @click="emit('add-to-cart', product)"
        >
          {{ quantityInCart > 0 ? `У кошику (${quantityInCart}) +` : 'У кошик' }}
        </button>
      </div>
    </div>
  </article>
</template>