<template>
  <el-dialog
    title="添加"
    :visible="addDialogVisible"
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
          <el-button @click="closeAddDialog">取 消</el-button>
          <el-button type="primary" @click="submitEditForm('addForm')">保 存</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    name: 'Add',
    props: {
      addDialogVisible: {
        type: Boolean,
        required: true
      },
      eduList: {
        type: Array,
        required: true
      },
      editRules: {
        type: Object,
        required: true
      }
    },
    data() {
      return {
        addForm: {  //添加表单的对象
          username: '',
          password: '',
          age: '',
          borth: '',
          edu: '',
          aboutFile: []
        }
      }
    },
    //页面加载方法
    mounted() {
    },
    methods: {
      //添加附件对象上传成功callback
      addAboutFileSuccess(res, file) {
        this.addForm.aboutFile.push(file);
        this.$refs.addAboutFile.clearValidate();
      },
      //添加附件对象删除成功callback   file:当前删除的附件对象  fileList:删除后剩余的附件集合
      addAboutFileRemove(file, fileList) {
        this.addForm.aboutFile = fileList;
      },
      //监听添加表单Dialog关闭event
      addDialogClose() {
        this.$refs["addForm"].resetFields();
        this.$emit("closeAddDialog", false);
      },
      //编辑表单提交事件
      submitEditForm(formName) {
        let _this = this;
        this.$refs[formName].validate((valid) => {
          if (valid) {
            console.log("add");
            console.log(_this.addForm);
          } else {
            return false;
          }
        });
      },
      //关闭添加模态事件
      closeAddDialog() {
        //调用父组件的closeAddDialog属性方法
        this.$emit("closeAddDialog", false);
      }
    }

  }

</script>


<style scoped>
  #addTable {
    width: 100%;
  }
</style>
