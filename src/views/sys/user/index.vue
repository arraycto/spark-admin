<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.username" placeholder="账户" style="width: 200px;" class="filter-item" />
      <el-input placeholder="用户真实名" style="width: 200px;" class="filter-item" />
      <el-select
        v-model="status"
        placeholder="用户状态"
        clearable
        class="filter-item"
        style="width: 130px"
      >
        <el-option
          v-for="item in statusOptions"
          :key="item.key"
          :label="item.display_name+'('+item.key+')'"
          :value="item.key"
        />
      </el-select>
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >查询</el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px;"
        type="success"
        icon="el-icon-edit"
        @click="handleCreate"
      >新增</el-button>
    </div>
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="ID">
        <template slot-scope="scope">{{ scope.$index }}</template>
      </el-table-column>
      <el-table-column label="昵称" align="center">
        <template slot-scope="scope">{{ scope.row.nickname }}</template>
      </el-table-column>
      <el-table-column label="账号" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.username }}</span>
        </template>
      </el-table-column>
      <el-table-column label="邮箱" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.email }}</span>
        </template>
      </el-table-column>
      <el-table-column label="性别" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.sex | sexFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center">
        <template slot-scope="scope">{{ scope.row.createTime }}</template>
      </el-table-column>
      <el-table-column class-name="status-col" label="状态" align="center">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status | typeFilter">{{ scope.row.status | statusFilter }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button v-if="row.status=='1'" size="mini" type="warning" @click="handleModifyStatus(row,'deleted')">
            锁定
          </el-button>
          <el-button v-if="row.isDeleted!='1'" size="mini" type="danger" @click="handleModifyStatus(row,'deleted')">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.pages"
      :limit.sync="listQuery.size"
      @pagination="getList"
    />
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="账号" prop="username">
          <el-input v-model="temp.username" />
        </el-form-item>
        <el-form-item label="真实姓名" prop="nickname">
          <el-input v-model="temp.nickname" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="temp.email" />
        </el-form-item>
        <el-form-item label="性别" prop="sex">
          <el-select v-model="temp.sex" class="filter-item" placeholder="Please select">
            <el-option v-for="item in sexOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="temp.status" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date" />
        </el-form-item>
        <el-form-item label="Remark">
          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea" placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          确定
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import waves from '@/directive/waves' // waves directive
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import { listData } from '@/api/sys/user.js'

const statusOptions = [
  { key: 0, display_name: '禁用' },
  { key: 1, display_name: '正常' },
  { key: 2, display_name: '锁定' },
  { key: 3, display_name: '过期' }
]

const sexOptions = [
  { key: 0, display_name: '女' },
  { key: 1, display_name: '男' }
]

const statusKeyValue = statusOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

const sexKeyValue = sexOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'User',
  components: { Pagination },
  directives: { waves },
  filters: {
    typeFilter(status) {
      const typeMap = {
        0: 'info',
        1: 'success',
        2: 'danger',
        3: 'warning'
      }
      return typeMap[status]
    },
    sexFilter(sex) {
      return sexKeyValue[sex]
    },
    statusFilter(type) {
      return statusKeyValue[type]
    }
  },
  data() {
    return {
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        pages: 1,
        size: 20,
        username: ''
      },
      statusOptions,
      sexOptions,
      status: '',
      dialogFormVisible: false,
      dialogStatus: '',
      temp: {
        id: undefined,
        username: '',
        nickname: '',
        email: '',
        status: '',
        sex: 1
      },
      textMap: {
        update: '编辑',
        create: '创建'
      },
      rules: {
        username: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }]
      }
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      listData(this.listQuery).then(response => {
        this.list = response.data.records
        this.total = response.data.total
        this.listQuery.pages = response.data.pages
        this.listQuery.size = response.data.size
        this.listLoading = false
      })
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        username: '',
        nickname: '',
        email: '',
        status: '',
        sex: 1
      }
    },
    handleFilter() {
      this.listQuery.pages = 1
      this.getList()
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    handleModifyStatus(row, status) {
      console.info('Create')
    }
  }
}
</script>