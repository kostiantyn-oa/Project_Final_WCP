<script setup lang="ts">
import { h, resolveComponent, type Ref } from 'vue'
import type { TableColumn } from '@nuxt/ui'
import { getPaginationRowModel } from '@tanstack/vue-table'

interface UTableInstance {

  tableApi: {
    getState: () => { pagination: { pageIndex: number; pageSize: number } }
    getFilteredRowModel: () => { rows: any[] }
    setPageIndex: (index: number) => void
    setColumnFilters: (filters: { id: string; value: string }[]) => void
  }
}

useHead({
  title: 'Список продуктів'
})

const UBadge = resolveComponent('UBadge')
const UButton = resolveComponent('UButton')

const colorMode = useColorMode()
const toggleTheme = () => {
  colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'
}

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

const { data, status } = await useFetch<{ products: Product[] }>('https://dummyjson.com/products', {
  key: 'products-list',
  transform: (data) => {
    return data || { products: [] }
  },
  lazy: true
})

const headerButtonProps = {
  color: 'neutral' as const,
  variant: 'ghost' as const,
  class: '-mx-2.5 dark:text-white'
}

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        ...headerButtonProps,
        label: 'Назва',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    filterFn: 'includesString',
    cell: ({ row }) => {
      return h('div', { class: 'dark:text-white' }, row.getValue('title'))
    }
  },
  {
    accessorKey: 'description',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        ...headerButtonProps,
        label: 'Опис',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      return h('div', { class: 'whitespace-normal dark:text-white' }, row.getValue('description'))
    },
    filterFn: 'includesString'
  },
  {
    accessorKey: 'price',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h('div', { class: 'text-right dark:text-white' }, [
        h(UButton, {
          ...headerButtonProps,
          label: 'Ціна',
          icon: isSorted
              ? isSorted === 'asc'
                  ? 'i-lucide-arrow-up-narrow-wide'
                  : 'i-lucide-arrow-down-wide-narrow'
              : 'i-lucide-arrow-up-down',
          onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
        })
      ])
    },
    cell: ({ row }) => {
      const price = Number.parseFloat(row.getValue('price'))
      const formatted = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      }).format(price)
      return h('div', { class: 'text-right font-medium dark:text-white' }, formatted)
    }
  },
  {
    accessorKey: 'rating',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        ...headerButtonProps,
        label: 'Оцінка',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      const rating = row.getValue('rating') as number
      const color = rating < 4.5 ? 'error' : 'success'
      return h(UBadge, { class: 'capitalize', variant: 'subtle', color }, () => rating)
    }
  },
  {
    accessorKey: 'brand',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        ...headerButtonProps,
        label: 'Бренд',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      return h('div', { class: 'dark:text-white' }, row.getValue('brand'))
    }
  },
  {
    accessorKey: 'category',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        ...headerButtonProps,
        label: 'Категорія',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      return h('div', { class: 'dark:text-white' }, row.getValue('category'))
    }
  },
  {
    accessorKey: 'thumbnail',
    header: () => {
      return h('div', { class: 'dark:text-white' }, 'Фото')
    },
    cell: ({ row }) => {
      return h('img', {
        src: row.getValue('thumbnail'),
        alt: row.original.title,
        class: 'w-[80px] h-[80px] object-cover'
      })
    }
  }
]

const table = useTemplateRef('table') as Ref<UTableInstance | null>
const pagination = ref({
  pageIndex: 0,
  pageSize: 5
})
const titleFilter = ref<string>('')
const descriptionFilter = ref<string>('')

const sorting = ref([])

const pageIndex = computed((): number => table.value?.tableApi?.getState().pagination.pageIndex || 0)
const pageSize = computed((): number => table.value?.tableApi?.getState().pagination.pageSize || 5)
const totalRows = computed((): number => table.value?.tableApi?.getFilteredRowModel().rows.length || 0)
const showingStart = computed((): number => pageIndex.value * pageSize.value + 1)
const showingEnd = computed((): number => Math.min((pageIndex.value + 1) * pageSize.value, totalRows.value))
const paginationText = computed((): string => `Показ ${showingStart.value}–${showingEnd.value} із ${totalRows.value} продуктів`)

watch([titleFilter, descriptionFilter], () => {
  if (table.value?.tableApi) {
    table.value.tableApi.setColumnFilters([
      { id: 'title', value: titleFilter.value },
      { id: 'description', value: descriptionFilter.value }
    ])
  }
})
</script>

<template>
  <div class="flex flex-col w-full p-4 space-y-4 pb-12">
    <div class="flex justify-between items-center">
      <h1 class="text-2xl font-bold text-center flex-1 dark:text-white">Список продуктів</h1>
      <UButton
          :icon="colorMode.value === 'dark' ? 'i-heroicons-sun-20-solid' : 'i-heroicons-moon-20-solid'"
          color="gray"
          variant="ghost"
          aria-label="Toggle theme"
          class="dark:text-white"
          @click="toggleTheme"
      />
    </div>
    <div class="flex px-4 py-3.5 border-b border-gray-200 dark:border-gray-700 w-full gap-4">
      <UInput v-model="titleFilter" class="flex-1 dark:text-white" placeholder="Пошук за назвою..." />
      <UInput v-model="descriptionFilter" class="flex-1 dark:text-white" placeholder="Пошук за описом..." />
    </div>

    <UTable
        ref="table"
        v-model:pagination="pagination"
        v-model:sorting="sorting"
        :data="data?.products"
        :columns="columns"
        :loading="status === 'pending'"
        :pagination-options="{
        getPaginationRowModel: getPaginationRowModel()
      }"
        class="flex-1 border border-gray-200 dark:border-gray-700"
    />

    <div class="flex justify-between items-center border-t border-gray-200 dark:border-gray-700 pt-4">
      <div class="flex-1"></div>
      <div class="flex justify-center">
        <UPagination
            :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
            :items-per-page="table?.tableApi?.getState().pagination.pageSize"
            :total="table?.tableApi?.getFilteredRowModel().rows.length"
            @update:page="(p: number) => table?.tableApi?.setPageIndex(p - 1)"
        />
      </div>
      <span class="text-sm text-gray-500 dark:text-white flex-1 text-right">{{ paginationText }}</span>
    </div>
  </div>
</template>

<style scoped>
th:not(:last-child) {
  border-right: 1px solid #e5e7eb;
}
.dark th:not(:last-child) {
  border-right: 1px solid #374151;
}

td, th {
  padding: 0.25rem;
}

td, th {
  font-size: 0.875rem;
}

.dark [data-ui='input'] input,
.dark [data-ui='input'] input::placeholder {
  color: #ffffff !important;
}

.dark .pagination button {
  color: #ffffff !important;
}
</style>