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
            <el-date-picker type="date" placeholder="选择日期" v-model="searchForm.beginBorth" size="small" :picker-options="beginBorthBefore"></el-date-picker>
          </el-form-item>
          <span>-&nbsp;</span>
          <el-form-item prop="endBorth">
            <el-date-picker type="date" placeholder="选择日期" v-model="searchForm.endBorth" size="small" :picker-options="endBorthAfter"></el-date-picker>
          </el-form-item>
        </el-form-item>
        <el-form-item prop="edu" label="学历">
          <el-select v-model="searchForm.edu" filterable placeholder="请选择学历" size="small">
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
      <el-dialog
        title="添加"
        :visible.sync="addDialogVisible"
        :center="true"
        @close="addDialogClose"
      >
        <el-form :model="addForm" :rules="editRules" ref="addForm">
          <table id="addTable">
            <tr>
              <td>
                <el-form-item label="用户名" label-width="80px" prop="username">
                  <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
                </el-form-item>
              </td>
              <td>
                <el-form-item label="密码" label-width="80px" prop="password">
                  <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td>
                <el-form-item label="年龄" label-width="80px" prop="age">
                  <el-input type="number" v-model.number="addForm.age" placeholder="请输入年龄"></el-input>
                </el-form-item>
              </td>
              <td>
                <el-form-item label="出生日期" label-width="80px" prop="borth">
                  <el-date-picker type="date" placeholder="选择出生日期" v-model="addForm.borth"
                                  style="width: 100%"></el-date-picker>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="学历" label-width="80px" prop="edu">
                  <el-select v-model="addForm.edu" filterable placeholder="请选择学历" style="width: 100%">
                    <el-option
                      v-for="item in eduList"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value">
                    </el-option>
                  </el-select>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="附件" label-width="80px" prop="aboutFile" ref="addAboutFile">
                  <el-upload
                    action="https://jsonplaceholder.typicode.com/posts/"
                    :on-success="addAboutFileSuccess"
                    :on-remove="addAboutFileRemove"
                    :file-list="addForm.aboutFile"
                    multiple>
                    <el-button size="small" type="primary">点击上传</el-button>
                  </el-upload>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="备注" label-width="80px" prop="comment">
                  <el-input type="textarea" :autosize="{minRows:5}" v-model="addForm.comment"
                            placeholder="请输入备注"></el-input>
                </el-form-item>
              </td>
            </tr>
          </table>
        </el-form>

        <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitEditForm('addForm','add')">保 存</el-button>
      </span>
      </el-dialog>

    </div>

    <!--查看详情模态框-->
    <div id="viewDetailDialog">
      <el-dialog
        title="查看详情"
        :visible.sync="viewDialogVisible"
        width="50%"
        :center="true"
      >
        <el-form>
          <table id="viewTable">
            <tr>
              <td class="viewKey">用户名</td>
              <td class="viewVal">{{viewForm.username}}</td>
              <td class="viewKey">密码</td>
              <td class="viewVal">{{viewForm.password}}</td>
            </tr>
            <tr>
              <td class="viewKey">年龄</td>
              <td class="viewVal">{{viewForm.age}}</td>
              <td class="viewKey">出生日期</td>
              <td class="viewVal">{{viewForm.borth}}</td>
            </tr>
            <tr>
              <td class="viewKey">学历</td>
              <td class="viewVal" colspan="3">{{viewForm.edu}}</td>
            </tr>
            <tr>
              <td class="viewKey">附件</td>
              <td class="viewVal" colspan="3">
                <el-upload
                  :file-list="viewForm.aboutFile"
                  multiple>
                </el-upload>
              </td>
            </tr>
            <tr>
              <td class="viewKey">备注</td>
              <td class="viewVal" colspan="3">1</td>
            </tr>
          </table>


        </el-form>

        <span slot="footer" class="dialog-footer"></span>
      </el-dialog>

    </div>
    <!--修改模态框-->
    <div id="updateDialog">
      <el-dialog
        title="修改"
        :visible.sync="updateDialogVisible"
        width="50%"
        :center="true"
        @close="updateDialogClose"
      >
        <el-form :model="updateForm" :rules="editRules" ref="updateForm">
          <table id="updateTable">
            <tr>
              <td>
                <el-form-item label="用户名" label-width="80px" prop="username">
                  <el-input v-model="updateForm.username" placeholder="请输入用户名"></el-input>
                </el-form-item>
              </td>
              <td>
                <el-form-item label="密码" label-width="80px" prop="password">
                  <el-input v-model="updateForm.password" placeholder="请输入密码"></el-input>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td>
                <el-form-item label="年龄" label-width="80px" prop="age">
                  <el-input type="number" v-model.number="updateForm.age" placeholder="请输入年龄"></el-input>
                </el-form-item>
              </td>
              <td>
                <el-form-item label="出生日期" label-width="80px" prop="borth">
                  <el-date-picker type="date" placeholder="选择出生日期" v-model="updateForm.borth"
                                  style="width: 100%"></el-date-picker>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="学历" label-width="80px" prop="edu">
                  <el-select v-model="updateForm.edu" filterable placeholder="请选择学历" style="width: 100%">
                    <el-option
                      v-for="item in eduList"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value">
                    </el-option>
                  </el-select>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="附件" label-width="80px" prop="aboutFile" ref="updateAboutFile">
                  <el-upload
                    action="https://jsonplaceholder.typicode.com/posts/"
                    :on-success="updateAboutFileSuccess"
                    :on-remove="updateAboutFileRemove"
                    :file-list="updateForm.aboutFile"
                    multiple>
                    <el-button size="small" type="primary">点击上传</el-button>
                  </el-upload>
                </el-form-item>
              </td>
            </tr>
            <tr>
              <td colspan="2">
                <el-form-item label="备注" label-width="80px" prop="comment">
                  <el-input type="textarea" :autosize="{minRows:5}" v-model="updateForm.comment"
                            placeholder="请输入备注"></el-input>
                </el-form-item>
              </td>
            </tr>
          </table>
        </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="updateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitEditForm('updateForm','update')">保 存</el-button>
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
        token: "44fd080323ab4441bd51847eae572012",
        addDialogVisible: false,  //添加模态框是否可见
        viewDialogVisible: false, //查看模态框是否可见
        updateDialogVisible: false,//修改模态框是否可见
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
        addForm: {  //添加表单的对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        },
        updateForm: {  //修改表单的对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        },
        viewForm:{  //查看详情的表单对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        },
        editRules: {//编辑时表单的校验规则对象
          username: [{required: true, message: '请输入用户名', trigger: 'blur'}],
          password: [{required: true, message: '请输入密码', trigger: 'blur'}],
          age: [{required: true, message: '请输入年龄', trigger: 'blur'}],
          borth: [{required: true, message: '请输入出生日期', trigger: 'blur'}],
          edu: [{required: true, message: '请选择学历', trigger: 'blur'}],
          aboutFile: [{required: true, message: '请选择附件', trigger: 'change'}]
        },
        //搜索栏中的开始时间日期范围过滤
        beginBorthBefore:{
          disabledDate: (time) => {
            let beginDateVal =  this.searchForm.endBorth;
            if (beginDateVal) {
              return time.getTime() > beginDateVal;
            }
          }
        },
        //搜索栏中的结束时间日期范围过滤
        endBorthAfter:{
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
      //添加附件对象上传成功callback
      addAboutFileSuccess(res, file) {
        this.addForm.aboutFile.push(file);
        this.$refs.addAboutFile.clearValidate();
      },
      //添加附件对象删除成功callback   file:当前删除的附件对象  fileList:删除后剩余的附件集合
      addAboutFileRemove(file, fileList) {
        this.addForm.aboutFile = fileList;
      },
      //修改附件对象上传成功callback
      updateAboutFileSuccess(res, file) {
        this.updateForm.aboutFile.push(file);
        this.$refs.updateAboutFile.clearValidate();
      },
      //修改附件对象删除成功callback  file:当前删除的附件对象  fileList:删除后剩余的附件集合
      updateAboutFileRemove(file, fileList) {
        this.updateForm.aboutFile = fileList;
      },
      //编辑click事件
      handleEdit(row) {
        this.updateDialogVisible = true;
        let options = {
          url: StringConstants.SERVER_URL + "/sdkTest/getSdkTestById",
          params: {id:row.id}
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.updateForm=handleData;
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
          params: {id:row.id}
        };
        this.axios.post(options.url, JSON.stringify(options.params), {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then((response) => {
          let {status, data} = response;
          if (status === 200) {
            let handleData = data.data;
            this.viewForm=handleData;
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
        this.$refs[formName].resetFields();
        let defaultParams = {
          current:this.tableCurrent,
          size:this.tableCurrentSize,
          token: this.token
        };
        this.getDataList(defaultParams);
      },
      /**
       * 提交编辑form
       * @param formName  表单的ref名称
       * @param operaName 操作类型名称(add->添加,update->修改)
       */
      submitEditForm(formName, operaName) {
        let _this = this;
        this.$refs[formName].validate((valid) => {
          if (valid) {
            console.log(operaName);
            console.log(_this.addForm);
          } else {
            return false;
          }
        });
      },
      //添加表单Dialog关闭event
      addDialogClose() {
        this.$refs["addForm"].resetFields();
      },
      //修改表单Dialog关闭event
      updateDialogClose() {
        this.$refs["updateForm"].resetFields();
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

  #addTable, #updateTable {
    width: 100%;
  }

  #viewTable{
    width: 100%;
  }


  .viewKey{
    height: 30px;
    width: 15%;
    text-align: center;
  }

  .viewVal{
    height: 30px;
    width: 40%;
    text-align: left;
  }


  #viewTable tr{
    height: 60px;
  }
</style>
