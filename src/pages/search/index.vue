<script lang="ts" setup>
import type { PaginationInfo } from 'naive-ui/es/pagination'
import { h, defineComponent } from 'vue'
import { NButton, useMessage } from 'naive-ui'
import type { DataTableColumns } from 'naive-ui'
const router = useRouter()
const route = useRoute()
const message = useMessage()

const loading = ref<boolean>(false)
const searchParams = reactive({
  queryKeyword: '',
  searchType: '0',
})
type Item = {
  domain: string
  record_id: string
  record_bureau: string
  record_time: string
}
const renderData = ref([])
const pagination = reactive<Pagination>({
  page: Number(route.query.page) || 1,
  pageSize: 15,
  itemCount: 0,
  prefix: (info: PaginationInfo) => {
    return h(
      'div',
      {},
      `共 ${info.itemCount} 条记录 - 当前页 ${renderData.value.length} 条`
    )
    // 解决 naiveui 带来的问题：只有一条记录时，endIndedx=1(startIndex=0)
  },
  onUpdatePage: async (page: number) => {
    pagination.page = page
    await fetchData()
    router.push({ query: { page } })
  },
})
const createColumns = ({
  play,
}: {
  play: (row: Item) => void
}): DataTableColumns<Item> => {
  return [
    {
      title: '域名',
      key: 'domain',
    },
    {
      title: '公安备案',
      key: 'record_time',
      render(row) {
        return h(
          NButton,
          {
            strong: true,
            secondary: true,
            onClick: () => play(row),
          },
          { default: () => '查看详情' }
        )
      },
    },
    {
      title: 'ICP 备案',
      key: 'record_id',
    },
    {
      title: '开办主体',
      key: 'record_bureau',
    },
    {
      title: '操作',
      key: 'actions',
      render(row) {
        return h(
          NButton,
          {
            strong: true,
            tertiary: true,
            size: 'small',
            onClick: () => play(row),
          },
          { default: () => '查看详情' }
        )
      },
    },
  ]
}

const columns = createColumns({
  play(row: Item) {
    message.info(`Play ${row.title}`)
  },
})

const fetchData = async () => {
  loading.value = true
  console.log('############', route.params.queryKeyword)
  searchParams.queryKeyword = route.params.queryKeyword as string
  searchParams.searchType = route.params.searchType as string
  try {
    const { data } = await axios.post(
      'api/search',
      {
        ...searchParams,
      },
      {
        params: {
          page: pagination.page,
          rows: pagination.pageSize,
        },
      }
    )
    console.log(data)
    pagination.itemCount = data.total
    renderData.value = data.list
  } catch (err) {
    // message
  } finally {
    loading.value = false
  }
}
fetchData()
</script>

<template>
  <div class="animation">
    <div flex="~" justify="between" align="items-baseline" text="white">
      <div flex="~" items="center">
        <b text="4xl" font="semibold">数据检索</b>
      </div>
      <!-- <p>
        数据更新：{{ formatDiffNow(0) }}
      </p> -->
    </div>
    <n-card
      title="高级检索"
      :bordered="false"
      footer-class="flex justify-end gap-4"
      class="my-10 shadow-md"
    >
      <template #footer>
        <n-button size="small">重置</n-button>
        <n-button size="small" type="primary">检索</n-button>
      </template>
    </n-card>
    <n-card
      :bordered="false"
      footer-class="flex justify-end gap-4"
      class="my-10 shadow-md"
    >
      <n-data-table
        :columns="columns"
        :data="renderData"
        :pagination="pagination"
        :bordered="false"
      />
    </n-card>
  </div>
</template>
