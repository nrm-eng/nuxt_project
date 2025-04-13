<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'

interface Product {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const { data: products, error } = await useFetch<{ products: Product[] }>('https://dummyjson.com/products', {
  query: { limit: 100 }
})

const columns: TableColumn<Product>[] = [
  { accessorKey: 'title', header: 'Назва', enableSorting: true },
  { accessorKey: 'description', header: 'Опис', enableSorting: true },
  {
    accessorKey: 'price',
    header: 'Ціна',
    enableSorting: true,
    cell: ({ row }) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(row.getValue('price'))
  },
  {
    accessorKey: 'rating',
    header: 'Оцінка',
    enableSorting: true,
    cell: ({ row }) => {
      const rating = row.getValue('rating') as number
      return h('span', {
        style: rating < 4.5 ? 'color: red;' : 'color: green;'
      }, rating.toFixed(1))
    }
  },
  { accessorKey: 'brand', header: 'Бренд', enableSorting: true },
  { accessorKey: 'category', header: 'Категорія', enableSorting: true },
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    cell: ({ row }) => h('img', {
      src: row.getValue('thumbnail'),
      alt: row.getValue('title'),
      style: 'width: 100px; height: 100px; object-fit: cover;'
    })
  }
]

const pageIndex = ref(0)
const pageSize = ref(10)
const searchQuery = ref('')
const sort = ref<{ key: string; direction: 'asc' | 'desc' } | null>(null)

watch(searchQuery, () => {
  pageIndex.value = 0
})

const filteredData = computed(() => {
  let result = products.value?.products || []

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(item =>
        Object.values(item).some(val => String(val).toLowerCase().includes(query))
    )
  }

  if (sort.value) {
    result = [...result].sort((a, b) => {
      const aValue = a[sort.value!.key as keyof Product]
      const bValue = b[sort.value!.key as keyof Product]
      if (aValue < bValue) return sort.value!.direction === 'asc' ? -1 : 1
      if (aValue > bValue) return sort.value!.direction === 'asc' ? 1 : -1
      return 0
    })
  }

  return result
})

const paginatedData = computed(() => {
  const start = pageIndex.value * pageSize.value
  return filteredData.value.slice(start, start + pageSize.value)
})

const pageCount = computed(() => Math.ceil(filteredData.value.length / pageSize.value))

const onSort = (key: string) => {
  if (sort.value?.key === key) {
    sort.value = {
      key,
      direction: sort.value.direction === 'asc' ? 'desc' : 'asc'
    }
  } else {
    sort.value = { key, direction: 'asc' }
  }
  pageIndex.value = 0
}

const selectRandomProduct = () => {
  const allProducts = products.value?.products || []
  if (allProducts.length > 0) {
    const randomIndex = Math.floor(Math.random() * allProducts.length)
    searchQuery.value = allProducts[randomIndex].title
  }
}
</script>

<template>
  <div class="space-y-4">
    <div class="search-wrapper">
      <UInput
          v-model="searchQuery"
          placeholder="Пошук за назвою..."
          class="max-w-md"
      />
      <div class="button-group">
        <button class="clear-button" @click="searchQuery = ''">
          <i class="i-lucide-refresh-cw"></i>
          Очистити
        </button>
      </div>
    </div>

    <UTable
        ref="table"
        :data="paginatedData"
        :columns="columns"
        sticky
        class="table h-96"
        @sort="onSort"
    />

    <div v-if="error" class="text-red-500">
      Error loading products: {{ error.message }}
    </div>
    <div class="pagination">
      <button
          :disabled="pageIndex === 0"
          @click="pageIndex = pageIndex - 1"
      >
        Попередня
      </button>
      <span>Сторінка {{ pageIndex + 1 }} з {{ pageCount }}</span>
      <button
          :disabled="pageIndex + 1 === pageCount"
          @click="pageIndex = pageIndex + 1"
      >
        Наступна
      </button>
    </div>
  </div>
</template>