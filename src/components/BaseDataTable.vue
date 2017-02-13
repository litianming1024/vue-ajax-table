<template>
  <div>
    <el-row>
      <el-col :span="innerActionsDef.width" :offset="innerActionsDef.offset" v-if="actionsShow">
        <action-bar :actions="innerActionsDef.def"></action-bar>
      </el-col>
      <el-col :span="innerSearchDef.width" :offset="innerSearchDef.offset" v-if="innerSearchDef.show">
        <el-input v-model="searchKey" icon="search" :on-icon-click="handleIconClick"></el-input>
      </el-col>
    </el-row>
    <el-table
      :data="tableData" @sort-change="handleSortChange" border="border" fit="fit"
      @row-click="handleRowClick" @selection-change="handleSelectionChange" style="width: 100%">
      <el-table-column v-if="innerCheckBoxDef.show" prop="innerCheckBox"
        type="selection"
        :width="innerCheckBoxDef.width">
      </el-table-column>
        <el-table-column
          :label="field.label" :prop="field.prop" :width="field.width" v-for="field in fields"
          :sortable="field.sortable">
        </el-table-column>
      <el-table-column
        label="操作" prop="innerRowActions" inline-template="inline-template" v-if="rowActionsShow"
        :min-width="actionColWidth">
        <div><span v-for="action in rowActionsDef">
          <el-button :type="action.type" @click="action.handler(row)">{{action.name}}</el-button></span></div>
      </el-table-column>
    </el-table>
    <div>
      <el-pagination
        @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="currentPage" :page-sizes="innerPaginationDef.pageSizes"
        :page-size="internalPageSize" :layout="innerPaginationDef.layout"
        :total="total"></el-pagination>
    </div>
  </div>
</template>
<script>
  import 'element-ui/lib/theme-default/index.css'
  import ActionBar from './ActionBar'
  export default {
    components: {
      ActionBar
    },
    props: {
      fields: {
        type: Array,
        required: true
      },
      apiUrl: {
        type: String,
        default: ''
      },
      actionsDef: {
        type: Object,
        default () {
          return {}
        }
      },
      searchDef: {
        type: Object,
        default () {
          return {}
        }
      },
      checkBoxDef: {
        type: Object,
        default () {
          return {}
        }
      },
      rowActionsDef: {
        type: Array,
        default () {
          return []
        }
      },
      actionColWidth: String,
      colNotRowClick: {
        type: Array,
        default () {
          return []
        }
      },
      paginationDef: {
        type: Object,
        default () {
          return {}
        }
      },
      loadOnStart: {
        type: Boolean,
        default: true
      },
      httpOptions: {
        type: Object,
        default: function () {
          return {}
        }
      },
      queryParams: {
        type: Object,
        default: function () {
          return {
            sort: 'sort',
            order: 'order',
            page: 'page',
            pageSize: 'pagesize',
            searchKey: 'search'
          }
        }
      }
    },
    data: function () {
      return {
        sort: '',
        order: '',
        currentPage: 1,
        internalPageSize: 20,
        searchKey: '',
        tableData: [],
        total: 0
      }
    },
    created: function () {
      if (this.loadOnStart) {
        this.loadData()
      }
      this.$parent.$on('refresh', this.refresh)
    },
    computed: {
      innerActionsDef () {
        return Object.assign({}, {
          def: [],
          width: 5,
          offset: 0
        }, this.actionsDef)
      },
      innerSearchDef () {
        return Object.assign({}, {
          show: true,
          props: undefined,
          filterFunction: undefined,
          width: 5,
          offset: 0
        }, this.searchDef)
      },
      innerPaginationDef () {
        return Object.assign({}, {
          layout: 'prev, pager, next, jumper, sizes, total',
          pageSize: 20,
          pageSizes: [1, 2, 3],
          currentPage: 1
        }, this.paginationDef)
      },
      innerRowActionsDef () {
        return Object.assign({}, {
          def: [],
          width: 5,
          offset: 0
        }, this.rowActionsDef)
      },
      innerColNotRowClick () {
        return this.colNotRowClick.concat(['innerRowActions', 'innerCheckBox'])
      },
      innerCheckBoxDef () {
        return Object.assign({}, {
          show: true,
          width: 55
        }, this.checkBoxDef)
      },
//      total () {
//        return 100
//        return this.tableData.length
//      },
      actionsShow () {
        return this.innerActionsDef.def.length > 0
      },
      rowActionsShow () {
        return this.innerRowActionsDef.def.length > 0
      }
    },
    methods: {
//      formatProps (props) {
//        return props ? [].concat(props) : undefined
//      },
      handleSortChange (column, prop, order) {
        console.debug(column)
        this.sort = column.prop
        this.order = column.order
//        this.queryParams.sort = column.prop
//        this.sortData = obj
        this.loadData()
      },
      handleSizeChange (size) {
        this.internalPageSize = size
        this.loadData()
      },
      handleCurrentChange (currentPage) {
        this.currentPage = currentPage
        this.loadData()
      },
      handleRowClick (row, event, column) {
        if (this.innerColNotRowClick.indexOf(column.property) === -1) {
          this.$emit('row-click', row)
        }
      },
      loadData: function (success = this.loadSuccess, failed = this.loadFailed) {
        this.httpOptions['params'] = this.getAllQueryParams()

        this.$http.get(this.apiUrl, this.httpOptions).then(
          success,
          failed
        )
      },
      loadSuccess: function (response) {
        let body = response.data
        this.tableData = body.data
        this.total = body.total
//        console.debug(body.data)
//        console.debug(this.data)
//        this.atableData = body
//        this.tableData = this.getObjectValue(body, this.dataPath, null)
      },
      loadFailed: function (response) {
      },
      getAllQueryParams: function () {
        let params = {}
//        params[this.queryParams.sort] = this.getSortParam()
        params[this.queryParams.pageSize] = this.internalPageSize
        params[this.queryParams.page] = this.currentPage
        params[this.queryParams.sort] = this.sort
        params[this.queryParams.order] = this.order
        params[this.queryParams.searchKey] = this.searchKey
//        for (let x in this.appendParams) {
//          params[x] = this.appendParams[x]
//        }

        return params
      },
      reload: function () {
        this.loadData()
      },
      handleSelectionChange (val) {
        // 将多选的数据向上传递
        this.$emit('selection-change', val)
      },
      refresh () {
        console.debug('refresh accept')
        this.loadData()
      },
      handleIconClick (ev) {
        console.log(ev)
        this.loadData()
      }
    }
  }
</script>
