<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="8" :sm="24">
              <a-form-item label="id">
                <a-input-number v-model="queryParam.id" style="width: 100%"/>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="使用状态">
                <a-select v-model="queryParam.status" placeholder="请选择" default-value="1">
                  <a-select-option value="%">全部</a-select-option>
                  <a-select-option value="0">停用</a-select-option>
                  <a-select-option value="1">启用</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <template v-if="advanced">
              <a-col :md="8" :sm="24">
                <a-form-item label="名称">
                  <a-input v-model="queryParam.name" style="width: 100%"/>
                </a-form-item>
              </a-col>
            </template>
            <a-col :md="!advanced && 8 || 24" :sm="24">
              <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
                <a-button type="primary" @click="loadTopAttachmentCat()">查询</a-button>
                <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">重置</a-button>
                <a @click="toggleAdvanced" style="margin-left: 8px">
                  {{ advanced ? '收起' : '展开' }}
                  <a-icon :type="advanced ? 'up' : 'down'"/>
                </a>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-operator">
        <a-button type="primary" icon="plus" @click="handleAdd">新建</a-button>
        <a-dropdown v-action:edit v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1"><a-icon type="delete" />删除</a-menu-item>
            <!-- lock | unlock -->
            <a-menu-item key="2"><a-icon type="lock" />锁定</a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px">
            批量操作 <a-icon type="down" />
          </a-button>
        </a-dropdown>
      </div>

      <a-table
        ref="table"
        :columns="columns"
        :data-source="topAttachmentCat"
        :loading="loading"
        :rowKey="record=>record.id"
        class="components-table-demo-nested">
        <span slot="method" slot-scope="method">
          <a-tag v-for="tag in method" :key="tag" color="blue">{{ tag }}</a-tag>
        </span>
        <span slot="status" slot-scope="status">
          <a-badge :status="status | statusTypeFilter" :text="status | statusFilter" />
        </span>
        <span slot="action" slot-scope="text, record">
          <template>
            <a @click="handleEdit(record)">编辑</a>
            <a-divider type="vertical" />
            <a @click="handleSub(record)">删除</a>
          </template>
        </span>
      </a-table>

      <create-form
        ref="createModal"
        :visible="visible"
        :loading="confirmLoading"
        :model="mdl"
        @cancel="handleCancel"
        @ok="handleOk"
      />
    </a-card>
  </page-header-wrapper>
</template>

<script>
import moment from 'moment'
import { STable } from '@/components'
import { getAttachmentCatList, createAttachmentCatList, updateAttachmentCatList, deleteAttachmentCatList } from '@/api/attachmentCat'

import CreateForm from './modules/AttachmentCatCreateForm'

const columns = [
  { title: 'id', dataIndex: 'id', key: 'id', scopedSlots: { customRender: 'id' } },
  { title: '标题', dataIndex: 'name', key: 'label' },
  { title: '标记', dataIndex: 'slug', key: 'value' },
  { title: '状态', dataIndex: 'status', key: 'status', scopedSlots: { customRender: 'status' } },
  { title: '更新时间', dataIndex: 'updated_at', key: 'updated_at' },
  { title: '操作', key: 'action', scopedSlots: { customRender: 'action' } }
]

const statusMap = {
  false: {
    status: 'default',
    text: '停用'
  },
  true: {
    status: 'success',
    text: '启用'
  }
}

export default {
  name: 'TableList',
  components: {
    STable,
    CreateForm
  },
  data () {
    this.columns = columns
    return {
      topAttachmentCat: [],
      // create model
      visible: false,
      loading: false,
      confirmLoading: false,
      mdl: null,
      // 高级搜索 展开/关闭
      advanced: false,
      // 查询参数
      queryParam: {
      },
      // 加载数据方法 必须为 Promise 对象
      selectedRowKeys: [],
      selectedRows: []
    }
  },
  filters: {
    statusFilter (type) {
      return statusMap[type].text
    },
    statusTypeFilter (type) {
      return statusMap[type].status
    }
  },
  computed: {
    rowSelection () {
      return {
        selectedRowKeys: this.selectedRowKeys,
        onChange: this.onSelectChange
      }
    }
  },
  created () {
    this.loadTopAttachmentCat()
  },
  methods: {
    async loadTopAttachmentCat () {
      this.loading = true
      const that = this
      await getAttachmentCatList(this.queryParam)
        .then(res => {
          if (res.code !== 200) {
            that.$message.error(res.message)
            return []
          }
          that.topAttachmentCat = res.result.data
          this.loading = false
        })
    },
    handleAdd () {
      this.mdl = {
        'status': true
      }
      this.visible = true
    },
    handleEdit (record) {
      this.visible = true
      this.mdl = { ...record }
    },
    handleOk () {
      const form = this.$refs.createModal.form
      this.confirmLoading = true
      form.validateFields((errors, values) => {
        if (!errors) {
          console.log(values)
          if (values.id > 0) {
            // 修改 e.g.
            updateAttachmentCatList(values).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.loadTopAttachmentCat()

              res.code === 200 ? this.$message.success(res.message) : this.$message.error(res.message)
            })
          } else {
            // 新增
            createAttachmentCatList(values).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.loadTopAttachmentCat()

              res.code === 200 ? this.$message.success(res.message) : this.$message.error(res.message)
            })
          }
        } else {
          this.confirmLoading = false
        }
      })
    },
    handleCancel () {
      this.visible = false

      const form = this.$refs.createModal.form
      form.resetFields() // 清理表单数据（可不做）
    },
    handleSub (record) {
      const form = this.$refs.createModal.form
      const that = this
      this.$confirm({
        title: '你确定要删除项目吗?',
        content: '如果你删除了此项目，将不可恢复！',
        onOk () {
          deleteAttachmentCatList(record).then(res => {
            that.visible = false
            that.confirmLoading = false
            // 重置表单数据
            form.resetFields()
            // 刷新表格
            that.loadTopAttachmentCat()
            res.code === 200 ? that.$message.success(res.message) : that.$message.error(res.message)
          })
        },
        onCancel () {}
      })
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
    },
    resetSearchForm () {
      this.queryParam = {
        date: moment(new Date())
      }
    }
  }
}
</script>
