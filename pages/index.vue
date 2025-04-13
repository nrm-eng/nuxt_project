<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'

// Тип для даних
type Payment = {
  id: string
  date: string
  status: 'paid' | 'failed' | 'refunded'
  email: string
  amount: number
}

// Початкові дані
const data = ref<Payment[]>([
  { id: '4600', date: '2024-03-11T15:30:00', status: 'paid', email: 'james@example.com', amount: 594 },
  { id: '4599', date: '2024-03-11T10:10:00', status: 'failed', email: 'mia@example.com', amount: 276 },
  { id: '4598', date: '2024-03-11T08:50:00', status: 'refunded', email: 'william@example.com', amount: 315 },
  { id: '4597', date: '2024-03-10T19:45:00', status: 'paid', email: 'emma@example.com', amount: 529 },
  { id: '4596', date: '2024-03-10T15:55:00', status: 'paid', email: 'ethan@example.com', amount: 639 },
  { id: '4595', date: '2024-03-09T12:20:00', status: 'failed', email: 'olivia@example.com', amount: 412 },
  { id: '4594', date: '2024-03-09T09:15:00', status: 'paid', email: 'liam@example.com', amount: 750 },
  { id: '4593', date: '2024-03-08T17:40:00', status: 'refunded', email: 'sophia@example.com', amount: 199 },
  { id: '4592', date: '2024-03-08T14:25:00', status: 'paid', email: 'noah@example.com', amount: 820 },
  { id: '4591', date: '2024-03-07T20:10:00', status: 'failed', email: 'ava@example.com', amount: 345 },
  { id: '4590', date: '2024-03-07T11:50:00', status: 'paid', email: 'mason@example.com', amount: 480 },
  { id: '4589', date: '2024-03-06T16:30:00', status: 'refunded', email: 'isabella@example.com', amount: 265 },
  { id: '4588', date: '2024-03-06T10:00:00', status: 'paid', email: 'jacob@example.com', amount: 910 },
  { id: '4587', date: '2024-03-05T18:45:00', status: 'failed', email: 'charlotte@example.com', amount: 123 },
  { id: '4586', date: '2024-03-05T13:20:00', status: 'paid', email: 'elijah@example.com', amount: 672 },
  { id: '4585', date: '2024-03-04T22:15:00', status: 'refunded', email: 'amelia@example.com', amount: 388 },
  { id: '4584', date: '2024-03-04T09:30:00', status: 'paid', email: 'benjamin@example.com', amount: 555 },
  { id: '4583', date: '2024-03-03T15:10:00', status: 'failed', email: 'harper@example.com', amount: 290 },
  { id: '4582', date: '2024-03-03T12:55:00', status: 'paid', email: 'lucas@example.com', amount: 700 },
  { id: '4581', date: '2024-03-02T19:25:00', status: 'refunded', email: 'evelyn@example.com', amount: 432 },
])

const columns: TableColumn<Payment>[] = [
  { accessorKey: 'id', header: '#', enableSorting: true },
  {
    accessorKey: 'date',
    header: 'Date',
    enableSorting: true,
    cell: ({ row }) => new Date(row.getValue('date')).toLocaleDateString('en-US', { day: 'numeric', month: 'short' })
  },
  { accessorKey: 'status', header: 'Status', enableSorting: true },
  { accessorKey: 'email', header: 'Email', enableSorting: true },
  {
    accessorKey: 'amount',
    header: 'Amount',
    enableSorting: true,
    cell: ({ row }) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'EUR' }).format(row.getValue('amount'))
  },
]

const pageIndex = ref(0)
const pageSize = ref(10)

const searchQuery = ref('')

const sort = ref<{ key: string; direction: 'asc' | 'desc' } | null>(null)

const filteredData = computed(() => {
  let result = [...data.value]

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(item =>
        Object.values(item).some(val => String(val).toLowerCase().includes(query))
    )
  }

  if (sort.value) {
    result.sort((a, b) => {
      const aValue = a[sort.value!.key as keyof Payment]
      const bValue = b[sort.value!.key as keyof Payment]
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

</script>

<template>
  <div class="space-y-4">
    <UInput
        v-model="searchQuery"
        placeholder="Search..."
        icon="i-lucide-search"
        class="max-w-md"
    />

    <UTable
        :columns="columns"
        :data="paginatedData"
        :sort="{ key: sort?.key, direction: sort?.direction }"
        @sort="onSort"
    />

    <UPagination
        v-model="pageIndex"
        :page-count="pageCount"
        :total="filteredData.length"
        show-first
        show-last
    />
  </div>
</template>