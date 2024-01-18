<script lang="ts" setup>
const router = useRouter()

const queryKeyword = ref<string>('')
const searchType = ref<number>(0)

const onInputKeyup = async (event: KeyboardEvent) => {
  // 拦截回车事件
  if (event.target !== event.currentTarget) return
  if (event.shiftKey || event.key !== 'Enter') return
  event.stopPropagation()
  event.preventDefault()
  // 回车事件处理
  // ....
  router.push({
    name: 'Search',
    params: { queryKeyword: queryKeyword.value, searchType: searchType.value },
  })
}

const options = [
  {
    label: '全局检索',
    value: 0,
  },
  {
    label: '域名',
    value: 1,
  },
  {
    label: '备案号',
    value: 2,
  },
  {
    label: '单位',
    value: 3,
  },
  {
    label: '网站',
    value: 4,
  },
]
</script>

<template>
  <n-select
    v-model:value="searchType"
    :options="options"
    default-value="1"
    class="header-select"
  />
  <n-input
    w="!100"
    size="large"
    clearable
    v-model:value="queryKeyword"
    placeholder="输入关键字回车以检索"
    :input-props="{
      onKeyup: onInputKeyup,
    }"
  >
    <template #prefix>
      <n-icon>
        <icon-clarity-search-line />
      </n-icon>
    </template>
  </n-input>
</template>
