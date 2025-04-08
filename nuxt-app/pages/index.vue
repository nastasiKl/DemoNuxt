<script setup lang="ts">
import { getPaginationRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import { h, resolveComponent } from 'vue'

definePageMeta({
  title: 'Список продуктів'
})

useHead({
  title: 'Список продуктів'
})

const table = useTemplateRef('table')
const UBadge = resolveComponent('UBadge')

//візуальна штука для статусів
//const UBadge = resolveComponent('UBadge')

type ProductResponse = {
  products: Product[]
}

type Product = {
  title: string
  description: string
  price: number
  rating: number // < 4.5 red, else green
  brand: string
  category: string
  thumbnail: string //фото 100*100
}
const { data, status } = await useFetch('https://dummyjson.com/products', {
  key: 'table-products',
  transform: (response: ProductResponse): Product[] => {
    return response.products.map(product => ({
      ...product,
      brand: product.brand || 'No Brand',
      rating: product.rating
    }))
  },
  lazy: true
})



const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: 'Title'
  },
  {
    accessorKey: 'description',
    header: 'Description',
  },

  {
    accessorKey: 'price',
    header: 'Price'
  },
  {
    accessorKey: 'rating',
    header: 'Rating',
    cell: ({ row }) => {
      const rating = row.original.rating
      const color = rating < 4.5 ? 'error' : 'success'

      return h(resolveComponent('UBadge'), {
        variant: 'subtle',
        color: color
      }, () => rating.toFixed(1))
    }
  },
  {
    accessorKey: 'brand',
    header: 'Brand',
  },
  {
    accessorKey: 'category',
    header: 'Category'
  },
  {
    accessorKey: 'thumbnail',
    header: 'Photo',
    cell: ({ row }) => {
      const url = row.original.thumbnail
      return h('img', {
        src: url,
        width: 100,
        height: 100,
        style: 'object-fit: cover; border-radius: 8px'
      })
    }
  }
]

const globalFilter = ref('')

const pagination = ref({
  pageIndex: 0,
  pageSize: 7

})
const handleFilterChange = () => {
  pagination.value.pageIndex = 0
}

watch(globalFilter, handleFilterChange)
</script>

<template>
  <div class="w-full max-w-screen-lg mx-auto space-y-4 pb-4">
  <div class="flex px-4 py-3.5 border-b border-(--ui-border-accented)">
      <UInput v-model="globalFilter" class="max-w-sm" placeholder="Filter..." />
    </div>
    <UTable
        ref="table"
        v-model:global-filter="globalFilter"
        v-model:pagination="pagination"
        :data="data || []"
        :columns="columns"

        :pagination-options="{
        getPaginationRowModel: getPaginationRowModel()
      }"
        class="table-auto w-full text-left flex-1"
    />

    <div class="flex justify-center border-t border-(--ui-border) pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>

