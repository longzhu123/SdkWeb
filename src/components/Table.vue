<template>
  <div id="table-container">
    <!--表格搜索工具栏-->
    <div id="searchBar">
      <el-form :model="searchForm" :inline="true" ref="searchForm">
        <el-form-item prop="ip" label="ip">
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
      <el-button type="success" size="small" icon="el-icon-plus" @click="addDialogVisible=true">添加</el-button>
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

    <!--分页组件-->
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

    <!--添加模态框-->
    <div id="addDialog">
      <el-dialog
        title="添加"
        :visible.sync="addDialogVisible"
        center="true"
      >
        <el-form :model="addForm">
          <table id="addTable">
            <tr>
              <td>
                <el-form-item label="ip" label-width="70px">
                  <el-input v-model="addForm.ip" placeholder="请输入ip"></el-input>
                </el-form-item>
              </td>
              <td>
                <el-form-item label="操作内容" label-width="80px">
                  <el-input v-model="addForm.operContent" placeholder="请输入操作内容"></el-input>
                </el-form-item>
              </td>
            </tr>

            <tr>
              <td colspan="2">
                <el-form-item label="操作耗时" label-width="70px">
                  <el-input v-model="addForm.taskTimeSpan" placeholder="请输入操作耗时"></el-input>
                </el-form-item>
              </td>
            </tr>
          </table>
        </el-form>

        <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialogVisible = false">保 存</el-button>
      </span>
      </el-dialog>

    </div>

    <!--查看详情模态框-->
    <div id="viewDetailDialog">
      <el-dialog
        title="查看详情"
        :visible.sync="viewDialogVisible"
        width="50%"
        center="true"
      >
        <span>查看详情</span>
        <span slot="footer" class="dialog-footer">
        <el-button @click="viewDialogVisible=false">取 消</el-button>
      </span>
      </el-dialog>

    </div>
    <!--修改模态框-->
    <div id="updateDialog">
      <el-dialog
        title="修改"
        :visible.sync="updateDialogVisible"
        width="50%"
        center="true"
      >
        <span>修改</span>
        <span slot="footer" class="dialog-footer">
        <el-button @click="updateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateDialogVisible = false">保 存</el-button>
      </span>
      </el-dialog>

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
        token: "729ad6e7b52c40ad947672449fc3a6c1",
        addDialogVisible: false,
        viewDialogVisible: false,
        updateDialogVisible: false,
        addForm: {
          ip: '',
          operContent: '',
          taskTimeSpan: ''
        },
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
        this.tableCurrent = 1;
        this.searchForm.current = this.tableCurrent;
        this.searchForm.size = this.tableCurrentSize;
        let searchParams = this.searchForm;
        this.getDataList(searchParams);
      },
      //当前current改变event
      handleCurrentChange(val) {
        this.tableCurrent = val;
        this.searchForm.current = this.tableCurrent;
        this.searchForm.token = this.token;
        this.searchForm.size = this.tableCurrentSize;
        let searchParams = this.searchForm;
        this.getDataList(searchParams);
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
        this.updateDialogVisible = true;
      },
      //查看click事件
      handleView() {
        this.viewDialogVisible = true;
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
      },
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

  #addTable{
    width: 100%;
  }
</style>
