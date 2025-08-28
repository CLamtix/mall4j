<template>
  <div class="mod-user">
    <avue-crud
      ref="crudRef"
      :page="page"
      :data="dataList"
      :option="tableOption"
      @search-change="onSearch"
      @selection-change="selectionChange"
      @on-load="getDataList"
    >
      <template #pic="scope">
        <span
          v-if="scope.row.pic"
          class="avue-crud__img"
        >
          <el-icon><Document /></el-icon>
        </span>
        <span v-else>-</span>
      </template>

      <template #status="scope">
        <el-tag
          v-if="scope.row.status === 0"
          type="danger"
        >
          禁用
        </el-tag>
        <el-tag v-else>
          正常
        </el-tag>
      </template>

      <template #menu="scope">
        <el-button
          v-if="isAuth('admin:user:update')"
          type="primary"
          icon="el-icon-edit"
          @click.stop="onAddOrUpdate(scope.row.userId)"
        >
          编辑
        </el-button>
        <el-button
          v-if="isAuth('admin:user:update') && scope.row.status === 1"
          type="danger"
          @click.stop="changeStatus(scope.row.userId, 0)"
          round
        >
          关小黑屋
        </el-button>
        <el-button
          v-if="isAuth('admin:user:update') && scope.row.status === 0"
          type="success"
          @click.stop="changeStatus(scope.row.userId, 1)"
          round
        >
          解除禁用
        </el-button>
      </template>
    </avue-crud>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdateRef"
      @refresh-data-list="getDataList"
    />
  </div>
</template>

<script setup>
import { isAuth } from '@/utils'
import { tableOption } from '@/crud/user/user.js'
import AddOrUpdate from './add-or-update.vue'

const dataList = ref([])
const dataListLoading = ref(false)
const dataListSelections = ref([])
const addOrUpdateVisible = ref(false)
const page = reactive({
  total: 0, // 总页数
  currentPage: 1, // 当前页数
  pageSize: 10 // 每页显示多少条
})
/**
 * 获取数据列表
 */
const getDataList = (pageParam, params, done) => {
  dataListLoading.value = true
  http({
    url: http.adornUrl('/admin/user/page'),
    method: 'get',
    params: http.adornParams(
      Object.assign(
        {
          current: pageParam == null ? page.currentPage : pageParam.currentPage,
          size: pageParam == null ? page.pageSize : pageParam.pageSize
        },
        params
      )
    )
  })
    .then(({ data }) => {
      dataList.value = data.records
      page.total = data.total
      dataListLoading.value = false
      if (done) done()
    })
}

const addOrUpdateRef = ref(null)
/**
 * 新增 / 修改
 * @param id
 */
const onAddOrUpdate = (id) => {
  addOrUpdateVisible.value = true
  nextTick(() => {
    addOrUpdateRef.value?.init(id)
  })
}

/**
 * 条件查询
 */
const onSearch = (params, done) => {
  getDataList(page, params, done)
}

/**
 * 多选变化
 */
const selectionChange = (val) => {
  dataListSelections.value = val
}

/**
 * 禁用 / 解除禁用
 * @param id
 */
const changeStatus = (id, status) => {
  http({
    url: http.adornUrl('/admin/user/changeStatus'),
    method: 'post',
    data: http.adornData({
      userId: id,
      status: status
    })
  })
    .then(() => {
      getDataList()
    })
}
</script>
