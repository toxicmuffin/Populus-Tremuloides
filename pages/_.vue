<template>
  <div>
    <div class="product-container">
      <ProductGallery :images="product.image_gallery" />
      <div class="product-details">
        <div class="shop-container">
          <h1>{{ product.title }}</h1>
          <p class="short-description">{{ product.short_description }}</p>
          <p class="product-price">
            {{
              Intl.NumberFormat('en', {
                style: 'currency',
                currency: 'USD',
              }).format(product.product_price)
            }}
          </p>
          <!-- quantity -->
          <div class="quantity-container">
            <label for="quantity">Quantity</label>
            <div class="quantity-wrapper">
              <button
                aria-label="Decrement button"
                class="quantity-buttons decrement-button"
                @click="decrement()"
              ></button>
              <input
                id="quantity"
                v-model="quantity.value"
                type="number"
                min="1"
                value="1"
              />
              <button
                aria-label="Increment button"
                class="quantity-buttons increment-button"
                @click="increment()"
              ></button>
            </div>
          </div>
          <button
            aria-label="Add to cart"
            class="snipcart-add-item add-to-cart"
            :data-item-name="product.title"
            :data-item-id="product.product_id"
            :data-item-price="product.product_price"
            :data-item-url="`${$config.baseUrl}${path}`"
            :data-item-description="product.short_description"
            :data-item-image="product.product_image"
            :data-item-quantity="`${quantity.value}`"
            v-bind="customFields"
            :disabled="quantity.value <= 0"
          >
            Add to Cart
          </button>
        </div>
        <nuxt-content :document="product" />
      </div>
    </div>
    <ProductFeature :products="relatedProducts" />
  </div>
</template>

<script lang="ts">
export default {
  async asyncData({ $content, params, error }) {
    const path = `/${params.pathMatch || 'index'}`
    const [product] = await $content('products', { deep: true })
      .where({ path })
      .fetch()
    const relatedProducts = await $content('products', { deep: true })
      .where({ tags: { $contains: product.tags } })
      .without(['body'])
      .fetch()
    if (!product) {
      return error({ statusCode: 404, message: 'Page not found - ' + path })
    }

    return {
      product,
      path,
      relatedProducts,
    }
  },
  data() {
    return {
      quantity: {
        type: Number,
        value: 1,
      },
      decrement() {
        if (this.quantity.value === 1) return
        this.quantity.value--
      },
      increment() {
        this.quantity.value++
      },
    }
  },
  head() {
    return {
      title: 'Populus Tremuloides - ' + this.product.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.product.short_description,
        },
        // Open Graph
        { hid: 'og:title', property: 'og:title', content: this.product.title },
        {
          hid: 'og:description',
          property: 'og:description',
          content: this.product.short_description,
        },
        // Twitter Card
        {
          hid: 'twitter:title',
          name: 'twitter:title',
          content: this.product.title,
        },
        {
          hid: 'twitter:description',
          name: 'twitter:description',
          content: this.product.short_description,
        },
      ],
    }
  },
  computed: {
    customFields() {
      if (
        this.product.custom_fields === undefined ||
        this.product.custom_fields.length === 0
      ) {
        return
      }
      return this.product.custom_fields
        .map(({ title, required, options }) => ({
          name: title,
          required,
          options,
        }))
        .map(
          (x: { [s: string]: unknown } | ArrayLike<unknown>, index: number) =>
            Object.entries(x).map(([key, value]) => ({
              [`data-item-custom${
                index + 1
              }-${key.toString().toLowerCase()}`]: value,
            }))
        )
        .reduce((acc: string | any[], curr: any) => acc.concat(curr), [])
        .reduce((acc: any, curr: any) => ({ ...acc, ...curr }))
    },
  },
}
</script>

<style lang="scss" scoped>
@import '~/assets/styles/variables/mixins.scss';
.shop-container {
  margin: 2rem;
  display: flex;
  flex-flow: column nowrap;
  justify-content: space-between;
  align-items: center;
  text-align: center;
}
.quantity-container {
  width: 134px;
  margin: 1rem auto;
  .quantity-wrapper {
    position: relative;
    display: flex;
    flex-flow: row nowrap;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--border-color-secondary);
    .quantity-buttons {
      width: 2rem;
      margin: auto;
      height: 1.5rem;
      font-size: 1.5rem;
      display: flex;
      flex-flow: column nowrap;
      align-items: center;
      justify-content: center;
    }
    .decrement-button {
      border-right: 1px solid var(--border-color-secondary);
      &.decrement-button::before {
        content: '';
        height: 2px;
        width: 20px;
        display: block;
        background-color: var(--color);
      }
    }
    .increment-button {
      border-left: 1px solid var(--border-color-secondary);
      &.increment-button::before,
      &.increment-button::after {
        content: '';
        position: absolute;
        height: 2px;
        width: 20px;
        display: block;
        background-color: var(--color);
      }
    }
    .increment-button::after {
      content: '';
      position: absolute;
      height: 2px;
      width: 20px;
      display: block;
      background-color: var(--color);
      transform: rotate(90deg);
    }
  }
}
input {
  width: 3rem;
  height: 2rem;
  padding: 0.25rem;
  font-size: 1.5rem;
  text-align: center;
  color: var(--color);
  background-color: var(--bg);
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type='number'] {
  -moz-appearance: textfield;
}
.product-container {
  @include medium {
    display: grid;
    grid-template-columns: 1fr 1fr;
    max-height: 90vh;
    margin-bottom: 2rem;
    border-bottom: 0.25rem solid var(--border-color);
    .product-details {
      overflow: scroll;
      max-height: 80vh;
      -ms-overflow-style: none;
      scrollbar-width: none;
      &::-webkit-scrollbar {
        display: none;
      }
    }
  }
  @include large {
    grid-template-columns: 2fr 1fr;
  }
  img {
    object-fit: cover;
    height: 100%;
    width: 100%;
    border-bottom: 0.25rem solid var(--border-color);
    @include medium {
      border-right: 0.25rem solid var(--border-color);
      border-bottom: none;
    }
  }
  h1 {
    font-size: 2rem;
  }
  p {
    &.short-description {
      font-size: 1.25rem;
    }
    &.product-price {
      font-size: 1.5rem;
    }
  }
  .add-to-cart {
    width: 100%;
    max-width: 134px;
    display: block;
    background-color: var(--button-color);
    color: var(--button-text);
    border-radius: 0.5rem;
    padding: 1rem 2rem;
  }
}
</style>
