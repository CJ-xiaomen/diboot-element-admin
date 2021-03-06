<template>
  <div class="app-container">

    <div class="table-page-search-wrapper">
      <el-form :inline="true" label-width="100px">
        <el-row :gutter="18">
          <el-col :md="8" :sm="24">
            <el-form-item label="姓名" style="width: 100%;">
              <el-input v-model="queryParam.realname" placeholder="姓名" style="width: 200px;" class="filter-item" @keyup.enter.native="onSearch" />
            </el-form-item>
          </el-col>
          <el-col :md="8" :sm="24">
            <el-form-item label="工号" style="width: 100%;">
              <el-input v-model="queryParam.userNum" placeholder="工号" style="width: 200px;" class="filter-item" @keyup.enter.native="onSearch" />
            </el-form-item>
          </el-col>
          <template v-if="advanced">
            <el-col :md="8" :sm="24">
              <el-form-item label="性别" style="width: 100%;">
                <el-select v-model="queryParam.gender" placeholder="请选择性别" style="width: 100%;">
                  <el-option
                    v-for="kv in more.genderKvList"
                    :key="kv.v"
                    :value="kv.v"
                    :label="kv.k"
                  />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :md="8" :sm="24">
              <el-form-item label="电话" style="width: 100%;">
                <el-input v-model="queryParam.mobilePhone" placeholder="电话" style="width: 200px;" class="filter-item" @keyup.enter.native="onSearch" />
              </el-form-item>
            </el-col>
            <el-col :md="8" :sm="24">
              <el-form-item label="邮箱" style="width: 100%;">
                <el-input v-model="queryParam.email" placeholder="邮箱" style="width: 200px;" class="filter-item" @keyup.enter.native="onSearch" />
              </el-form-item>
            </el-col>
          </template>
          <el-col :md="!advanced && 8 || 24" :sm="24">
            <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
              <el-button v-waves type="primary" icon="el-icon-search" @click="onSearch">
                查询
              </el-button>
              <el-button style="margin-left: 8px" type="info" icon="el-icon-refresh" @click="reset">
                重置
              </el-button>
              <el-link type="primary" :underline="false" style="margin-left: 8px" @click="toggleAdvanced">
                {{ advanced ? '收起' : '展开' }}
                <i :class="advanced ? 'el-icon-arrow-up' : 'el-icon-arrow-down'" />
              </el-link>
            </span>
          </el-col>
        </el-row>
      </el-form>
    </div>
    <div class="table-operator">
      <el-button v-permission="['create']" class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-plus" @click="$refs.form.open(undefined)">
        新建
      </el-button>
      <el-button v-permission="['import']" class="filter-item" style="margin-left: 10px;" type="default" icon="el-icon-upload2" @click="$refs.userImport.open()">
        批量导入
      </el-button>
      <el-button v-permission="['export']" class="filter-item" style="margin-left: 10px;" type="default" :icon="exportLoadingData ? 'el-icon-loading' : 'el-icon-download'" @click="exportData">
        导出
      </el-button>
    </div>
    <el-table
      v-loading="loadingData"
      :data="list"
      element-loading-text="Loading"
      fit
      highlight-current-row
      row-key="id"
      @sort-change="appendSorterParam"
    >
      <el-table-column label="部门">
        <template slot-scope="scope">
          {{ scope.row.orgShortName }}
        </template>
      </el-table-column>
      <el-table-column label="姓名">
        <template slot-scope="scope">
          {{ scope.row.realname }}
        </template>
      </el-table-column>
      <el-table-column label="用户编号">
        <template slot-scope="scope">
          {{ scope.row.userNum }}
        </template>
      </el-table-column>
      <el-table-column label="性别">
        <template slot-scope="scope">
          {{ scope.row.genderLabel }}
        </template>
      </el-table-column>
      <el-table-column label="电话" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.mobilePhone }}</span>
        </template>
      </el-table-column>
      <el-table-column label="邮箱" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.email }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button
            v-permission="['detail']"
            type="text"
            @click="$refs.detail.open(row.id)"
          >
            详情
          </el-button>
          <span
            v-permission="['detail']"
            v-permission-again="['position', 'update', 'delete']"
          >
            <el-divider
              direction="vertical"
            />
          </span>
          <el-button
            v-permission="['position']"
            type="text"
            @click="$refs.userPositionRefForm.open(row)"
          >
            岗位
          </el-button>
          <span
            v-permission="['position']"
            v-permission-again="['update', 'delete']"
          >
            <el-divider
              direction="vertical"
            />
          </span>
          <el-dropdown
            v-permission="['update', 'delete']"
            @command="command => menuCommand(command, row)"
          >
            <el-button type="text">
              更多<i class="el-icon-arrow-down el-icon--right" />
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                v-permission="['update']"
                command="update"
                icon="el-icon-edit"
              >
                更新
              </el-dropdown-item>
              <el-dropdown-item
                v-permission="['delete']"
                command="delete"
                icon="el-icon-delete"
              >
                删除
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <span v-permission-missing="['detail', 'update', 'delete']">
            -
          </span>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="pagination.total>0"
      :total="pagination.total"
      :page.sync="pagination.current"
      :limit.sync="pagination.pageSize"
      :style="{textAlign: 'right'}"
      @pagination="handlePaginationChanged"
    />
    <user-position-ref-form ref="userPositionRefForm" />
    <user-detail ref="detail" />
    <user-form
      ref="form"
      :current-node-id="currentNodeId"
      @complete="getList"
    />
    <user-import ref="userImport" :current-node-id="currentNodeId" @complete="getList" />
  </div>
</template>
<script>
import userDetail from './userDetail'
import userForm from './userForm'
import waves from '@/directive/waves'
import list from '@/components/diboot/mixins/list'
import userImport from '@/views/orgStructure/orgUser/userImport'
import userPositionRefForm from '@/views/orgStructure/orgUser/userPositionRefForm'

export default {
  name: 'OrgUserList',
  components: {
    userDetail,
    userForm,
    userImport,
    userPositionRefForm
  },
  directives: { waves },
  mixins: [list],
  props: {
    currentNodeId: {
      type: String,
      default: '0'
    }
  },
  data() {
    return {
      baseApi: '/iam/user',
      exportApi: '/excel/export',
      getMore: true
    }
  },
  watch: {
    currentNodeId: function(val) {
      if (!val || val === '0' || val === 0) {
        this.customQueryParam = {}
      } else {
        this.customQueryParam = { orgId: val }
      }
      this.getList()
    }
  },
  methods: {
    openUserPositionRefForm(record) {
      this.$refs.userPositionRefForm.open(record)
    }
  }
}
</script>
