<template>
<el-container>
  <task-sidebar></task-sidebar>
  <el-main>
    <el-form :inline="true">
      <el-form-item :label="t('task.id')">
        <el-input v-model.trim="searchParams.id" style="width: 180px;"></el-input>
      </el-form-item>
      <el-form-item :label="t('task.name')">
        <el-input v-model.trim="searchParams.name" style="width: 180px;"></el-input>
      </el-form-item>
      <el-form-item :label="t('task.tag')">
        <el-input v-model.trim="searchParams.tag" style="width: 180px;"></el-input>
      </el-form-item>
      <br>
      <el-form-item :label="t('task.protocol')">
        <el-select v-model.trim="searchParams.protocol" style="width: 180px;">
          <el-option :label="t('select')" value=""></el-option>
          <el-option
            v-for="item in protocolList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label="t('task.taskNode')">
        <el-select v-model.trim="searchParams.host_id" style="width: 180px;">
          <el-option :label="t('select')" value=""></el-option>
          <el-option
            v-for="item in hosts"
            :key="item.id"
            :label="item.alias + ' - ' + item.name + ':' + item.port "
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label="t('common.status')">
        <el-select v-model.trim="searchParams.status" style="width: 180px;">
          <el-option :label="t('select')" value=""></el-option>
          <el-option
            v-for="item in statusList"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="search()">{{ t('common.search') }}</el-button>
      </el-form-item>
    </el-form>
    <el-row type="flex" justify="end" style="margin-bottom: 10px;">
      <el-col :span="24" style="text-align: right;">
        <span v-if="isAdmin && selectedTasks.length > 0" style="margin-right: 10px; color: #909399;">已选择 {{ selectedTasks.length }} 个任务</span>
        <el-button v-if="isAdmin" type="success" size="default" @click="batchEnable" :disabled="selectedTasks.length === 0">批量启用</el-button>
        <el-button v-if="isAdmin" type="warning" size="default" @click="batchDisable" :disabled="selectedTasks.length === 0">批量禁用</el-button>
        <el-button v-if="isAdmin" type="danger" size="default" @click="batchRemove" :disabled="selectedTasks.length === 0">批量删除</el-button>
        <el-button type="primary" @click="toEdit(null)" v-if="isAdmin">{{ t('common.add') }}</el-button>
        <el-button type="info" @click="refresh">{{ t('common.refresh') }}</el-button>
      </el-col>
    </el-row>
    <el-pagination
      background
      layout="prev, pager, next, sizes, total"
      :total="taskTotal"
      v-model:current-page="searchParams.page"
      v-model:page-size="searchParams.page_size"
      @size-change="changePageSize"
      @current-change="changePage">
    </el-pagination>
    <el-table
      :data="tasks"
      tooltip-effect="dark"
      border
      @selection-change="handleSelectionChange"
      style="width: 100%">
      <el-table-column type="selection" width="55" v-if="isAdmin"></el-table-column>
      <el-table-column type="expand">
        <template #default="scope">
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="任务创建时间:">
              {{ $filters.formatTime(scope.row.created) }} <br>
            </el-form-item>
            <el-form-item label="任务类型:">
              {{ formatLevel(scope.row.level) }} <br>
            </el-form-item>
            <el-form-item label="单实例运行:">
               {{ formatMulti(scope.row.multi) }} <br>
            </el-form-item>
            <el-form-item label="超时时间:">
              {{ formatTimeout(scope.row.timeout) }} <br>
            </el-form-item>
            <el-form-item label="重试次数:">
              {{scope.row.retry_times}} <br>
            </el-form-item>
            <el-form-item label="重试间隔:">
              {{ formatRetryTimesInterval(scope.row.retry_interval) }}
            </el-form-item> <br>
            <el-form-item label="任务节点">
              <div v-for="item in scope.row.hosts" :key="item.host_id">
                {{item.alias}} - {{item.name}}:{{item.port}} <br>
              </div>
            </el-form-item> <br>
            <el-form-item label="命令:" style="width: 100%">
              {{scope.row.command}}
            </el-form-item> <br>
            <el-form-item label="备注" style="width: 100%">
              {{scope.row.remark}}
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column
        prop="id"
        :label="t('task.id')">
      </el-table-column>
      <el-table-column
        prop="name"
        :label="t('task.name')"
      width="150">
      </el-table-column>
      <el-table-column
        prop="tag"
        :label="t('task.tag')">
      </el-table-column>
      <el-table-column
        prop="spec"
        :label="t('task.cronExpression')"
      width="120">
      </el-table-column>
      <el-table-column :label="t('task.nextRunTime')" width="160">
        <template #default="scope">
          {{ $filters.formatTime(scope.row.next_run_time) }}
        </template>
      </el-table-column>
      <el-table-column
        prop="protocol"
        :formatter="formatProtocol"
        :label="t('task.protocol')">
      </el-table-column>
      <el-table-column
        :label="t('common.status')" v-if="isAdmin">
          <template #default="scope">
            <el-switch
              v-if="scope.row.level === 1"
              v-model="scope.row.status"
              :active-value="1"
              :inactive-value="0"
              active-color="#13ce66"
              @change="changeStatus(scope.row)"
              inactive-color="#ff4949">
            </el-switch>
          </template>
      </el-table-column>
      <el-table-column :label="t('common.status')" v-else>
        <template #default="scope">
          <el-switch
            v-if="scope.row.level === 1"
            v-model="scope.row.status"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            :disabled="true"
            inactive-color="#ff4949">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column :label="t('common.operation')" width="220" v-if="isAdmin">
        <template #default="scope">
          <el-row>
            <el-button type="primary" @click="toEdit(scope.row)">{{ t('common.edit') }}</el-button>
            <el-button type="success" @click="runTask(scope.row)">{{ t('task.manualRun') }}</el-button>
          </el-row>
          <br>
          <el-row>
            <el-button type="info" @click="jumpToLog(scope.row)">{{ t('task.viewLog') }}</el-button>
            <el-button type="danger" @click="remove(scope.row)">{{ t('common.delete') }}</el-button>
          </el-row>
        </template>
      </el-table-column>
    </el-table>
  </el-main>
</el-container>
</template>

<script>
import { useI18n } from 'vue-i18n'
import taskSidebar from './sidebar.vue'
import taskService from '../../api/task'
import { useUserStore } from '../../stores/user'
import { ElMessageBox } from 'element-plus'

export default {
  name: 'task-list',
  setup() {
    const { t } = useI18n()
    return { t }
  },
  data () {
    const userStore = useUserStore()
    return {
      tasks: [],
      hosts: [],
      taskTotal: 0,
      isFirstActivate: true,
      selectedTasks: [],
      searchParams: {
        page_size: 20,
        page: 1,
        id: '',
        protocol: '',
        name: '',
        tag: '',
        host_id: '',
        status: ''
      },
      isAdmin: userStore.isAdmin,
      protocolList: [
        {
          value: '1',
          label: 'http'
        },
        {
          value: '2',
          label: 'shell'
        }
      ],
      statusList: [
        {
          value: '2',
          label: '激活'
        },
        {
          value: '1',
          label: '停止'
        }
      ]
    }
  },
  components: {taskSidebar},
  created () {
    const hostId = this.$route.query.host_id
    if (hostId) {
      this.searchParams.host_id = hostId
    }

    this.search()
  },
  activated () {
    if (this.isFirstActivate) {
      this.isFirstActivate = false
      return
    }
    this.search()
  },
  methods: {
    formatLevel (value) {
      return value === 1 ? '主任务' : '子任务'
    },
    formatTimeout (value) {
      return value > 0 ? value + '秒' : '不限制'
    },
    formatRetryTimesInterval (value) {
      return value > 0 ? value + '秒' : '系统默认'
    },
    formatMulti (value) {
      return value > 0 ? '否' : '是'
    },
    changeStatus (item) {
      if (item.status) {
        taskService.enable(item.id, () => {
          this.search()
        })
      } else {
        taskService.disable(item.id, () => {
          this.search()
        })
      }
    },
    formatProtocol (row, col) {
      if (row[col.property] === 2) {
        return 'shell'
      }
      if (row.http_method === 1) {
        return 'http-get'
      }
      return 'http-post'
    },
    changePage (page) {
      this.searchParams.page = page
      this.search()
    },
    changePageSize (pageSize) {
      this.searchParams.page_size = pageSize
      this.search()
    },
    search (callback = null) {
      taskService.list(this.searchParams, (tasks, hosts) => {
        this.tasks = tasks.data
        this.taskTotal = tasks.total
        this.hosts = hosts
        if (callback) {
          callback()
        }
      })
    },
    runTask (item) {
      ElMessageBox.confirm(
        `确定要手动执行任务 "${item.name}" 吗？`,
        '手动执行任务',
        {
          confirmButtonText: '确定执行',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }
      ).then(() => {
        taskService.run(item.id, () => {
          this.$message.success('任务已开始执行')
        })
      }).catch(() => {})
    },
    remove (item) {
      ElMessageBox.confirm(
        `确定要删除任务 "${item.name}" 吗？`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        taskService.remove(item.id, () => {
          this.refresh()
        })
      }).catch(() => {})
    },
    jumpToLog (item) {
      this.$router.push(`/task/log?task_id=${item.id}`)
    },
    refresh () {
      this.search(() => {
        this.$message.success('刷新成功')
      })
    },
    toEdit (item) {
      let path = ''
      if (item === null) {
        path = '/task/create'
      } else {
        path = `/task/edit/${item.id}`
      }
      this.$router.push(path)
    },
    handleSelectionChange (selection) {
      this.selectedTasks = selection.filter(task => task.level === 1)
    },
    batchEnable () {
      if (this.selectedTasks.length === 0) {
        this.$message.warning('请选择要启用的任务')
        return
      }
      ElMessageBox.confirm(
        `确定要启用选中的 ${this.selectedTasks.length} 个任务吗？`,
        '批量启用任务',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        const ids = this.selectedTasks.map(task => task.id)
        taskService.batchEnable(ids, () => {
          this.$message.success('批量启用成功')
          this.selectedTasks = []
          this.search()
        })
      }).catch(() => {})
    },
    batchDisable () {
      if (this.selectedTasks.length === 0) {
        this.$message.warning('请选择要禁用的任务')
        return
      }
      ElMessageBox.confirm(
        `确定要禁用选中的 ${this.selectedTasks.length} 个任务吗？`,
        '批量禁用任务',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        const ids = this.selectedTasks.map(task => task.id)
        taskService.batchDisable(ids, () => {
          this.$message.success('批量禁用成功')
          this.selectedTasks = []
          this.search()
        })
      }).catch(() => {})
    },
    batchRemove () {
      if (this.selectedTasks.length === 0) {
        this.$message.warning('请选择要删除的任务')
        return
      }
      ElMessageBox.confirm(
        `确定要删除选中的 ${this.selectedTasks.length} 个任务吗？此操作不可恢复！`,
        '批量删除任务',
        {
          confirmButtonText: '确定删除',
          cancelButtonText: '取消',
          type: 'error'
        }
      ).then(() => {
        const ids = this.selectedTasks.map(task => task.id)
        taskService.batchRemove(ids, () => {
          this.$message.success('批量删除成功')
          this.selectedTasks = []
          this.search()
        })
      }).catch(() => {})
    }
  }
}
</script>
<style scoped>
  .demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>
