<template>
  <div class="login-container">
    <el-form
      class="login-form"
      autocomplete="on"
      :model="loginForm"
      :rules="loginRules"
      ref="loginForm"
      label-position="left"
    >
      <div class="title-container">
        <h3 class="title">{{ $t('login.title') }}</h3>
        <lang-select class="set-language"></lang-select>
      </div>
      <el-form-item prop="username">
        <span class="svg-container svg-container_login">
          <svg-icon icon-class="user" />
        </span>
        <el-input
          name="username"
          type="text"
          v-model="loginForm.username"
          autocomplete="on"
          :placeholder="$t('login.username')"
        />
      </el-form-item>

      <el-form-item prop="password">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input
          name="password"
          :type="passwordType"
          @keyup.enter.native="handleLogin"
          v-model="loginForm.password"
          autocomplete="on"
          :placeholder="$t('login.password')"
        />
        <span class="show-pwd" @click="showPwd">
          <svg-icon icon-class="eye" />
        </span>
      </el-form-item>

      <el-button
        type="primary"
        style="width:100%;margin-bottom:30px;"
        :loading="loading"
        @click.native.prevent="handleLogin"
      >{{ $t('login.logIn') }}</el-button>

      <el-button
        type="defaults"
        style="width:100%;margin-bottom:30px;margin-left:0"
        :loading="loading"
        @click.native.prevent="handleAdd"
      >{{ $t('login.resIn') }}</el-button>
    </el-form>

    <!-- // from dialog -->
    <el-dialog title="单位（项目）概况登记" :visible.sync="dialogFormVisible" width="70%" class="register">
      <el-form :model="editForm" :rules="editFormRules" ref="editForm" label-width="95px">
        <el-row>
          <el-col :span="8">
            <el-form-item label="用户名" prop="username">
              <el-input readonly="true" v-model="editForm.username" placeholder="请输入用户名"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item v-if="dialogStatus == 'create'" label="密码" prop="password">
              <el-input type="password" v-model="editForm.password" placeholder="请输入密码"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="角色" prop="roleId">
              <el-select
                class="filter-item"
                style="width: 225px"
                v-model="editForm.roleId"
                placeholder="请选择角色"
              >
                <el-option
                  v-for="item in editRolesOptions"
                  :key="item.roleId"
                  :label="item.roleDesc"
                  :value="item.roleId"
                  :disabled="false"
                >
                  <span style="float: left">{{ item.roleDesc }}</span>
                  <span style="float: right; color: #8492a6; font-size: 13px">{{ item.roleCode }}</span>
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <div style="width:100%;height:10px;background:#ccc;margin-bottom:10px"></div>

        <el-row>
          <el-col :span="8">
            <el-form-item label="单位名称" prop="company_bu">
              <el-input v-model="editForm.company_name" placeholder="请输入单位（项目）名称"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="主管单位" prop="company_name">
              <el-input v-model="editForm.company_bu" placeholder="请输入单位（项目）主管单位"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="单位地点" prop="company_addr">
              <el-input v-model="editForm.company_addr" placeholder="请输入单位（项目）地点"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="单位经纬度" prop="company_position">
              <el-input v-model="editForm.company_position" placeholder="请输入单位（项目部）经纬度"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="单位状态" prop="company_status">
              <el-select
                class="filter-item"
                style="width: 225px"
                v-model="editForm.company_status"
                placeholder="请选择单位状态"
                clearable
                filterable
              >
                <el-option
                  v-for="(item, index) in statusOptions"
                  :key="index"
                  :label="item | statusFilter"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="项目进度" prop="project_progress">
              <el-input v-model="editForm.company_position" placeholder="请输入项目进度（百分比）"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="探矿权人" prop="Tankuangquan">
              <el-input v-model="editForm.company_name" placeholder="请输入探矿权人"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="施工单位" prop="doCompany">
              <el-input v-model="editForm.company_bu" placeholder="请输入施工单位"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="到岗人数" prop="factNumber">
              <el-input v-model="editForm.company_addr" placeholder="请输入到岗人数"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="计划人数" prop="planNumber">
              <el-input v-model="editForm.company_position" placeholder="请输入计划人数"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="单位类型" prop="projectType">
              <el-select
                class="filter-item"
                style="width: 225px"
                v-model="editForm.company_status"
                placeholder="请选择单位类型"
                clearable
                filterable
              >
                <el-option
                  v-for="(item, index) in statusOptions2"
                  :key="index"
                  :label="item | statusFilterPT"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="合同价格" prop="price">
              <el-input v-model="editForm.company_position" placeholder="请输入合同价格"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <!--设计工作量目的任务-->
            <el-form-item label="单位概况" prop="projectIntroduction">
              <el-input
                type="textarea"
                style="width:546px"
                v-model="editForm.projectIntroduction"
                placeholder="请输入设计工作量目的任务（单位概况）"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item></el-form-item>
          </el-col>
          <el-col :span="8">
            <!--主要施工机具-->
            <el-form-item label="施工机具" prop="mainTools">
              <el-input
                type="textarea"
                style="width:225px"
                v-model="editForm.mainTools"
                placeholder="请输入主要施工机具"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <div style="width:100%;height:10px;background:#ccc;margin-bottom:10px"></div>

        <el-row>
          <el-col :span="8">
            <el-form-item label="项目负责人" prop="projectManager">
              <el-input v-model="editForm.projectManager" placeholder="请输入项目负责人"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="联系电话" prop="projectManagerTel">
              <el-input v-model="editForm.projectManagerTel" placeholder="请输入项目负责人联系电话"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item></el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="技术负责人" prop="projectTecManager">
              <el-input v-model="editForm.projectTecManager" placeholder="请输入项目技术负责人"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="类型" prop="projectTecManagerType">
              <el-select
                class="filter-item"
                style="width: 225px"
                v-model="editForm.projectTecManagerType"
                placeholder="请选择技术负责人类型"
                clearable
                filterable
              >
                <el-option
                  v-for="(item, index) in statusOptions3"
                  :key="index"
                  :label="item | statusFilterPTMT"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="联系电话" prop="projectTecManagerTel">
              <el-input v-model="editForm.projectManagerTel" placeholder="请输入项目技术负责人联系电话"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="安全员" prop="projectSafeManager">
              <el-input v-model="editForm.projectSafeManager" placeholder="请输入项目安全员"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="类型" prop="projectSafeManagerType">
              <el-select
                class="filter-item"
                style="width: 225px"
                v-model="editForm.projectTecManagerType"
                placeholder="请选择安全员类型"
                clearable
                filterable
              >
                <el-option
                  v-for="(item, index) in statusOptions3"
                  :key="index"
                  :label="item | statusFilterPTMT"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="联系电话" prop="projectSafeManagerTel">
              <el-input v-model="editForm.projectSafeManagerTel" placeholder="请输入项目安全员联系电话"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="8">
            <el-form-item label="开工日期" prop="projectStartDate">
              <el-date-picker
                v-model="editForm.projectStartDate"
                type="datetime"
                format="yyyy-MM-dd hh:mm"
                value-format="yyyy-MM-dd hh:mm"
                placeholder="选择开工日期"
              ></el-date-picker>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="完工日期" prop="projectEndDate">
              <el-date-picker
                v-model="editForm.projectEndDate"
                type="datetime"
                format="yyyy-MM-dd hh:mm"
                value-format="yyyy-MM-dd hh:mm"
                placeholder="选择计划完工日期"
              ></el-date-picker>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="附件上传" prop="file">
              <el-upload
                class="upload-demo"
                action="https://jsonplaceholder.typicode.com/posts/"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                :before-remove="beforeRemove"
                multiple
                :limit="3"
                :on-exceed="handleExceed"
                :file-list="fileList"
              >
                <el-button type="primary">点击上传</el-button>
                <div slot="tip" class="el-upload__tip">文件大小不超过10M</div>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>

        <div style="width:100%;height:10px;background:#ccc;margin-bottom:10px"></div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="formCancel('editForm')">取 消</el-button>
        <el-button v-if="dialogStatus == 'create'" type="primary" @click="handleLogin()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { isvalidUsername } from '@/utils/validate';
import LangSelect from '@/components/LangSelect';
import { fetchRoleList } from '@/api/role';

export default {
  components: { LangSelect },
  name: 'login',
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!isvalidUsername(value)) {
        callback(new Error(this.$t('tips.error_login_username')));
      } else {
        callback();
      }
    };
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error(this.$t('tips.error_login_password')));
      } else {
        callback();
      }
    };
    return {
      loginForm: {
        username: 'superAdmin',
        password: 'test1234'
      },
      loginRules: {
        username: [
          { required: true, trigger: 'blur', validator: validateUsername }
        ],
        password: [
          { required: true, trigger: 'blur', validator: validatePassword }
        ]
      },
      passwordType: 'password',
      loading: false,
      showDialog: false,
      dialogFormVisible: false, // 表单是否显示
      dialogStatus: '',
      editForm: {
        username: '',
        password: ''
      },
      editFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        company_bu: [
          { required: true, message: '请输入单位名称', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        company_name: [
          { required: true, message: '请输入主管单位', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        company_position: [
          { required: true, message: '请输入单位经纬度', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        company_addr: [
          { required: true, message: '请输入地址', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],

        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 10, message: '长度在 5 到 10 个字符', trigger: 'blur' }
        ],

        project_progress: [
          { required: true, message: '请输入项目进度', trigger: 'blur' },
          { min: 1, max: 10, message: '0~100', trigger: 'blur' }
        ],

        Tankuangquan: [
          { required: true, message: '请输入探矿权人', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],

        doCompany: [
          { required: false, message: '请输入施工单位', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],

        factNumber: [
          { required: true, message: '请输入到岗人数', trigger: 'blur' },
          { min: 1, max: 10, message: '0~100', trigger: 'blur' }
        ],

        planNumber: [
          { required: true, message: '请输入计划人数', trigger: 'blur' },
          { min: 1, max: 10, message: '0~100', trigger: 'blur' }
        ],

        projectType: [
          { required: true, message: '请选单位类型', trigger: 'blur' }
        ],

        price: [
          { required: true, message: '请输入合同金额', trigger: 'blur' },
          { min: 1, max: 10, message: '0~10000000', trigger: 'blur' }
        ],

        projectIntroduction: [
          {
            required: true,
            message: '请输入设计工作量，目的任务（单位概况）',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 20,
            message: '长度在100 到 2000 个字符',
            trigger: 'blur'
          }
        ],

        mainTools: [
          { required: true, message: '请输入主要施工机具', trigger: 'blur' },
          {
            min: 3,
            max: 20,
            message: '长度在100 到 2000 个字符',
            trigger: 'blur'
          }
        ],

        projectManager: [
          { required: true, message: '请输入项目负责人', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在2 到50 个字符', trigger: 'blur' }
        ],

        projectManagerTel: [
          {
            required: true,
            message: '请输入项目负责人联系电话',
            trigger: 'blur'
          },
          { min: 3, max: 20, message: '长度为11的数字', trigger: 'blur' }
        ],

        projectTecManager: [
          { required: true, message: '请输入项目技术负责人', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在2 到50 个字符', trigger: 'blur' }
        ],

        projectTecManagerType: [
          { required: true, message: '请选择', trigger: 'blur' }
        ],

        projectTecManagerTel: [
          {
            required: true,
            message: '请输入项目技术负责人联系电话',
            trigger: 'blur'
          },
          { min: 3, max: 20, message: '长度为11的数字', trigger: 'blur' }
        ],

        projectSafeManager: [
          { required: true, message: '请输入项目安全员', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在2 到50 个字符', trigger: 'blur' }
        ],

        projectSafeManagerType: [
          { required: true, message: '请选择', trigger: 'blur' }
        ],

        projectSafeManagerTel: [
          {
            required: true,
            message: '请输入项目安全员联系电话',
            trigger: 'blur'
          },
          { min: 3, max: 20, message: '长度为11的数字', trigger: 'blur' }
        ],

        projectStartDate: [
          { required: true, message: '请选择开工日期', trigger: 'blur' },
          { min: 3, max: 30, message: '时间', trigger: 'blur' }
        ],

        projectEndDate: [
          { required: true, message: '请选计划完工日期', trigger: 'blur' },
          { min: 3, max: 30, message: '时间', trigger: 'blur' }
        ],

        roleId: [{ required: true, message: '请选择角色', trigger: 'blur' }],

        company_status: [
          { required: true, message: '请选择单位状态', trigger: 'blur' }
        ]
      },
      editRolesOptions: [],
      statusOptions: [0, 1, 2],
      statusOptions2: [
        '金属非金属矿山',
        '选矿厂',
        '尾矿库',
        '使用危险化学品（细分：炸药库、实验室）',
        '学校',
        '建筑施工（细分：工程勘察、基础施工、检测监测、建安工程、装修工程、其他）',
        '地灾治理（细分：地灾评估、矿山治理、边坡治理、其他）',
        '市政工程（细分：道路桥梁、管网施工、隧道施工、地铁施工、垃圾场施工、水池施工、其他）',
        '机械制造（细分：生产、安装）',
        '地质勘查',
        '地理信息',
        '基地',
        '其他'
      ],
      statusOptions3: ['专项', '兼职'],
      fileList: []
    };
  },
  created() {
    this.getRoleList();
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = '';
      } else {
        this.passwordType = 'password';
      }
    },
    getRoleList() {
      fetchRoleList().then(res => {
        this.editRolesOptions = res.data;
      });
    },
    formCancel(_from) {
      this.dialogFormVisible = false;
    },
    formCreate() {
      this.$router.push({ path: '/user' });
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true;
          this.$store
            .dispatch('LoginByUsername', this.loginForm)
            .then(() => {
              this.loading = false;
              this.$router.push({ path: '/' });
            })
            .catch(() => {
              this.loading = false;
            });
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    formReset() {
      this.editForm = {
        username: '',
        password: '',
        statu: '',
        roleId: ''
      };
    },
    handleAdd() {
      this.formReset();
      this.dialogStatus = 'create';
      this.dialogFormVisible = true;
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${
          files.length
        } 个文件，共选择了 ${files.length + fileList.length} 个文件`
      );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    }
  },

  filters: {
    statusFilterPT(status2) {
      const statusMap2 = {
        0: '筹备2',
        1: '在建2',
        2: '停止2',
        3: '停止3'
      };
      return statusMap2[status2];
    },
    statusFilterPTMT(status3) {
      const statusMap3 = {
        0: '筹备22',
        1: '在建22',
        2: '停止22',
        3: '停止23'
      };
      return statusMap3[status3];
    },
    statusFilter(status) {
      const statusMap = {
        0: '筹备',
        1: '在建',
        2: '停止'
      };
      return statusMap[status];
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss">
$bg: #2d3a4b;
$light_gray: #eee;

/* reset element-ui css */
.login-form .el-input__inner {
  border: none !important;
}
</style>

<style rel="stylesheet/scss" lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

.login-container {
  position: fixed;
  height: 100%;
  width: 100%;
  background-color: $bg;

  .login-form {
    position: absolute;
    left: 0;
    right: 0;
    width: 520px;
    padding: 35px 35px 15px 35px;
    margin: 120px auto;

    .el-input {
      display: inline-block;
      height: 47px;
      width: 85%;
      input {
        background: transparent;
        border: 0px;
        -webkit-appearance: none;
        border-radius: 0px;
        padding: 12px 5px 12px 15px;
        color: $light_gray;
        height: 47px;
        &:-webkit-autofill {
          -webkit-box-shadow: 0 0 0px 1000px $bg inset !important;
          -webkit-text-fill-color: #fff !important;
        }
      }
    }
    .el-form-item {
      border: 1px solid rgba(25, 255, 255, 0.1);
      background: #fff;
      border-radius: 5px;
      color: #454545;
    }
  }
  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;
    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }
  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
    &_login {
      font-size: 20px;
    }
  }
  .title-container {
    position: relative;
    .title {
      font-size: 26px;
      font-weight: 400;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
    .set-language {
      color: #fff;
      position: absolute;
      top: 5px;
      right: 0px;
    }
  }
  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }
  .thirdparty-button {
    position: absolute;
    right: 35px;
    bottom: 28px;
  }
}
</style>
