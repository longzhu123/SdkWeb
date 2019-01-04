<template>
  <div id="table-container">
    <!--表格搜索工具栏-->
    <div id="searchBar">
      <el-form :model="searchForm" :inline="true" ref="searchForm">
        <el-form-item prop="username" label="用户名">
          <el-input placeholder="请输入用户名" v-model="searchForm.username" size="small"></el-input>
        </el-form-item>
        <el-form-item prop="password" label="密码">
          <el-input placeholder="请输入密码" v-model="searchForm.password" size="small"></el-input>
        </el-form-item>
        <el-form-item prop="age" label="年龄">
          <el-input placeholder="请输入年龄" type="number" v-model.number="searchForm.age" size="small"></el-input>
        </el-form-item>
        <el-form-item label="出生日期">
          <el-form-item prop="beginBorth">
            <el-date-picker type="date" placeholder="选择日期" v-model="searchForm.beginBorth" size="small"
                            :picker-options="beginBorthBefore"></el-date-picker>
          </el-form-item>
          <span>-&nbsp;</span>
          <el-form-item prop="endBorth">
            <el-date-picker type="date" placeholder="选择日期" v-model="searchForm.endBorth" size="small"
                            :picker-options="endBorthAfter"></el-date-picker>
          </el-form-item>
        </el-form-item>
        <el-form-item prop="edu" label="学历">
          <el-select v-model="searchForm.edu" filterable placeholder="请选择学历" size="small" class="selectEduMain">
            <el-option
              v-for="item in eduList"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
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
        prop="username"
        label="用户名"
        align="center"
        show-overflow-tooltip
      >
      </el-table-column>
      <el-table-column
        prop="password"
        label="密码"
        align="center"
        show-overflow-tooltip
      >
      </el-table-column>
      <el-table-column
        prop="age"
        label="年龄"
        align="center"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="borth"
        label="出生日期"
        align="center"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="edu"
        label="学历"
        align="center"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button
            size="small"
            icon="el-icon-view"
            type="primary"
            @click="handleView(scope.row)">查看
          </el-button>
          <el-button
            size="small"
            icon="el-icon-edit"
            type="primary"
            @click="handleEdit(scope.row)">修改
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
      <Add :addDialogVisible="addDialogVisible" :eduList="eduList" :editRules="editRules"
           @closeAddDialog="closeAddDialog"/>
    </div>

    <!--查看详情模态框-->
    <div id="viewDetailDialog">
      <ViewDetail :viewDialogVisible="viewDialogVisible" @closeViewDialog="closeViewDialog" :viewForm="viewForm"/>
    </div>
    <!--修改模态框-->
    <div id="updateDialog">
      <Update :updateDialogVisible="updateDialogVisible" @closeUpdateDialog="closeUpdateDialog" :updateObj="updateForm"
              :editRules="editRules" :eduList="eduList"/>
    </div>

  </div>

</template>

<script>
  import * as StringConstants from "../constants/StringConstants";
  import util from "../util/util";
  import Add from "./Add";
  import ViewDetail from "./View";
  import Update from "./Update";


  export default {
    name: 'Table',
    components: {Add, ViewDetail, Update},
    //存放全局数据
    data() {
      return {
        tableData: [],//表格数据对象
        tableCurrent: 1,//表格当前页
        tableCurrentSize: 10,//表格当前显示的记录数
        tableTotalSize: 0,//表格的总记录数
        tableSelection: [],//表格被选中的对象
        searchForm: {//搜索参数对象
          username:'',
          password:'',
          age:'',
          beginBorth:'',
          endBorth:'',
          edu:[]
        },
        token: "44fd080323ab4441bd51847eae572012",
        addDialogVisible: false,  //添加模态框是否可见
        viewDialogVisible: false, //查看模态框是否可见
        updateDialogVisible: false,//修改模态框是否可见
        updateForm: {  //修改表单的对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        },
        viewForm: {  //查看详情的表单对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        },
        eduList: [
          {   //学历下拉框数组对象
            value: '1',
            label: '小学'
          }, {
            value: '2',
            label: '初中'
          }, {
            value: '3',
            label: '高中'
          }, {
            value: '4',
            label: '大专'
          }, {
            value: '5',
            label: '本科'
          }],
        editRules: {//编辑时表单的校验规则对象
          username: [{required: true, message: '请输入用户名', trigger: 'blur'}],
          password: [{required: true, message: '请输入密码', trigger: 'blur'}],
          age: [{required: true, message: '请输入年龄', trigger: 'blur'}],
          borth: [{required: true, message: '请输入出生日期', trigger: 'blur'}],
          edu: [{required: true, message: '请选择学历', trigger: ['blur','change']}],
          aboutFile: [{required: true, message: '请选择附件', trigger: 'change'}]
        },
        //搜索栏中的开始时间日期范围过滤
        beginBorthBefore: {
          disabledDate: (time) => {
            let beginDateVal = this.searchForm.endBorth;
            if (beginDateVal) {
              return time.getTime() > beginDateVal;
            }
          }
        },
        //搜索栏中的结束时间日期范围过滤
        endBorthAfter: {
          disabledDate: (time) => {
            let beginDateVal = this.searchForm.beginBorth;
            if (beginDateVal) {
              return time.getTime() < beginDateVal;
            }
          }
        }
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
        let _this = this;
        let options = {
          url: StringConstants.SERVER_URL + "/sdkTest/likeSearchSdkTestByPage",
          params
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            _this.tableData = handleData.records;
            _this.tableCurrent = handleData.current;
            _this.tableTotalSize = handleData.total;
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
        this.searchForm.token = this.token;
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
            url: StringConstants.SERVER_URL + "/sdkTest/deleteBatchSdkTestByIds",
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
          url: StringConstants.SERVER_URL + "/sdkTest/likeSearchSdkTestByPage",
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
      handleEdit(row) {
        this.updateDialogVisible = true;
        let options = {
          url: StringConstants.SERVER_URL + "/sdkTest/getSdkTestById",
          params: {id: row.id}
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.updateForm = handleData;
          } else {
            alert("请求失败")
          }
        }).then().catch((e) => {
          console.error(e);
          alert("网络异常");
        });
      },
      //查看click事件
      handleView(row) {
        this.viewDialogVisible = true;
        let options = {
          url: StringConstants.SERVER_URL + "/sdkTest/getSdkTestById",
          params: {id: row.id}
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.viewForm = handleData;
          } else {
            alert("请求失败")
          }
        }).catch((e) => {
          console.error(e);
          alert("网络异常");
        });
      },
      //表格Checkbox选择事件
      handleSelectionChange(val) {
        this.tableSelection = val;
      },
      //重置搜索表单click事件
      resetForm(formName) {
        console.log(this.searchForm);
        this.$refs[formName].resetFields();
        let defaultParams = {
          current: this.tableCurrent,
          size: this.tableCurrentSize,
          token: this.token
        };
        console.log(this.searchForm);
        this.getDataList(defaultParams);
      },
      //关闭添加模态框
      closeAddDialog(val) {
        this.addDialogVisible = val;
      },
      //关闭查看详情模态框
      closeViewDialog(val) {
        this.viewDialogVisible = val;
      },
      //关闭修改模态框
      closeUpdateDialog(val) {
        this.updateDialogVisible = val;
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
</style>
