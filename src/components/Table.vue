<template>
  <div id="table-container">
    <!--表格搜索工具栏-->
    <div id="searchBar">
      <el-form :model="searchForm" :inline="true" ref="searchForm">
        <el-form-item prop="ip" ip="ip">
          <el-input placeholder="ip" v-model="searchForm.ip" size="small"></el-input>
        </el-form-item>
        <el-form-item prop="operContent" label="操作内容">
          <el-input placeholder="操作内容" v-model="searchForm.operContent" size="small"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="small" icon="el-icon-search" @click="search">搜索</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="small" icon="el-icon-refresh" @click="resetForm('searchForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!--表格操作工具栏-->
    <div id="tableOperaBar">
      <el-button type="success" size="small" icon="el-icon-plus">添加</el-button>
      <el-button type="danger" size="small" icon="el-icon-delete" @click="batchDelete">删除</el-button>
    </div>
    <el-table
      :data="tableData"
      height="620"
      tooltip-effect="dark"
      border
      style="width: 100%;"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        prop="ip"
        label="ip"
        align="center"
        show-overflow-tooltip
      >
      </el-table-column>
      <el-table-column
        prop="operContent"
        label="操作内容"
        align="center"
        show-overflow-tooltip
      >
      </el-table-column>
      <el-table-column
        prop="taskTimeSpan"
        label="操作耗时"
        align="center"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button
            size="small"
            icon="el-icon-view"
            type="primary"
            @click="handleView()">查看
          </el-button>
          <el-button
            size="small"
            icon="el-icon-edit"
            type="primary"
            @click="handleEdit()">修改
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="tableCurrent"
        :page-sizes="[10, 15, 20, 30,50]"
        :page-size="tableCurrentSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="tableTotalSize">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import * as StringConstants from "../constants/StringConstants";
  import util from "../util/util";

  export default {
    name: 'Table',
    //存放全局数据
    data() {
      return {
        tableData: [],//表格数据对象
        tableCurrent: 1,//表格当前页
        tableCurrentSize: 10,//表格当前显示的记录数
        tableTotalSize: 0,//表格的总记录数
        tableSelection: [],//表格被选中的对象
        searchForm: {},//搜索参数对象,
        token:"729ad6e7b52c40ad947672449fc3a6c1"
      }
    },
    //页面加载方法
    mounted() {
      let defaultParams = {
        current: StringConstants.DEFAULT_PAGE_CURRENT,
        size: StringConstants.PAGE_SIZE,
        token: this.token
      };
      this.getDataList(defaultParams);
    },
    methods: {
      /**
       * 分页获取数据列表
       * @param params 过滤参数对象
       */
      getDataList(params) {
        let options = {
          url: StringConstants.SERVER_URL + "/sysLogOperate/likeSearchSysLogOperateByPage",
          params
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.tableData = handleData.records;
            this.tableCurrent = handleData.current;
            this.tableTotalSize = handleData.total;
          } else {
            alert("请求失败")
          }
        }).catch((e) => {
          console.error(e);
          alert("网络异常");
        });
      },
      //分页组件Size改变event
      handleSizeChange(val) {
        this.tableCurrentSize = val;
        this.tableCurrent=1;
        let defaultParams = {
          current: this.tableCurrent,
          size: this.tableCurrentSize,
          token: this.token
        };
        let searchParams = Object.assign(defaultParams,this.searchForm);
        this.getDataList(searchParams);
      },
      //当前current改变event
      handleCurrentChange(val) {
        this.tableCurrent = val;
        let defaultParams = {
          current: this.tableCurrent,
          size: this.tableCurrentSize,
          token: this.token
        };
        this.getDataList(defaultParams);
      },
      //批量删除click事件
      batchDelete() {
        this.$confirm('确认删除吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'error'
        }).then(() => {
          let selectObj = this.tableSelection;
          let ids = util.convertArrayToFiledList(selectObj, "id");
          if (ids === undefined || ids === null || ids.length === 0) {
            this.$message({
              type: 'error',
              message: '请勾选记录,再删除!'
            });
            return;
          }
          let options = {
            url: StringConstants.SERVER_URL + "/sysLogOperate/deleteBatchSysLogOperateByIds",
            params: {"token": this.token, "ids": ids}
          };
          this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
            let {status} = response;
            if (status === 200) {
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
              let loadDataOptions = {
                current: this.tableCurrent,
                size: this.tableCurrentSize,
                token: this.token,
              };
              let searchOptions = Object.assign(loadDataOptions, this.searchForm);
              this.getDataList(searchOptions);
            } else {
              alert("请求失败")
            }
          }).catch((e) => {
            console.error(e);
            alert("网络异常");
          });

        }).catch(() => {
          //对话框取消click执行callback
        });


      },
      //搜索click事件
      search() {
        this.tableCurrent = 1;
        let searchForm = this.searchForm;
        let pageOptions = {
          current: this.tableCurrent,
          size: this.tableCurrentSize,
          token: this.token,
        };
        //搜索参数对象合并
        let searchOptions = Object.assign(searchForm, pageOptions);
        let options = {
          url: StringConstants.SERVER_URL + "/sysLogOperate/likeSearchSysLogOperateByPage",
          params: searchOptions
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.tableData = handleData.records;
            this.tableCurrent = handleData.current;
            this.tableTotalSize = handleData.total;
          } else {
            alert("请求失败")
          }
        }).catch((e) => {
          console.error(e);
          alert("网络异常");
        });
      },
      //编辑click事件
      handleEdit() {
        alert("编辑")
      },
      //查看click事件
      handleView() {
        alert("查看")
      },
      //表格Checkbox选择事件
      handleSelectionChange(val) {
        this.tableSelection = val;
      },
      //重置搜索表单click事件
      resetForm(formName) {
        this.$refs[formName].resetFields();
        let defaultParams = {
          current: StringConstants.DEFAULT_PAGE_CURRENT,
          size: StringConstants.PAGE_SIZE,
          token: this.token
        };
        this.getDataList(defaultParams);
      }
    }
  }
</script>

<style scoped>

  #table-container {
    margin-top: 20px;
  }

  #searchBar {
    margin-bottom: 10px;
  }

  #tableOperaBar {
    margin-bottom: 20px;
  }

  .block {
    text-align: right;
    margin-right: 30px;
    margin-top: 10px;
  }
</style>
