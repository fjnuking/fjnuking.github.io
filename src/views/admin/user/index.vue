<template>
  <div class="app-container calendar-list-container">
    <div class="filter-container">
      <el-input
        style="width: 200px;"
        class="filter-item"
        placeholder="用户名"
        v-model="listPageParams.username"
        @keyup.enter.native="handleSearch"
      ></el-input>
      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleSearch()">搜索</el-button>
      <el-button
        v-if="user_add"
        class="filter-item"
        style="margin-left: 10px;"
        type="primary"
        icon="edit"
        v-waves
        @click="handleAdd()"
      >添加</el-button>
    </div>
    <el-table
      :key="tableKey"
      :data="list"
      v-loading="listLoading"
      element-loading-text="数据加载中..."
      border
      fit
      highlight-current-row
      style="width: 100%"
      :default-sort="{ prop: 'userId', order: 'descending' }"
    >
      <el-table-column align="center" label="序号" sortable prop="userId">
        <template slot-scope="scope">{{ scope.row.userId }}</template>
      </el-table-column>

      <el-table-column align="center" label="用户名">
        <template slot-scope="scope">
          <span>
            <img
              v-if="scope.row.picUrl"
              class="user-picUrl"
              style="width: 20px; height: 20px; border-radius: 50%;"
              :src="scope.row.picUrl"
            />
            {{ scope.row.username }}
          </span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="单位名称">
        <template slot-scope="scope">
          <span>{{ scope.row.company_name }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="主管单位">
        <template slot-scope="scope">
          <span>{{ scope.row.company_bu }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="单位位置">
        <template slot-scope="scope">
          <span>{{ scope.row.company_position }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="单位地址">
        <template slot-scope="scope">
          <span>{{ scope.row.company_addr }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" class-name="status-col" label="单位状态">
        <template slot-scope="scope">
          <el-tag
            :type="
              scope.row.company_status == 0
                ? 'warning'
                : scope.row.company_status == 1
                ? 'scucess'
                : 'danger'
            "
          >{{ scope.row.company_status | statusFilter }}</el-tag>
        </template>
      </el-table-column>

      <el-table-column align="center" label="所属角色" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-for="(role, key) in scope.row.roleList" :key="key">{{ role.roleDesc }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="创建时间" sortable prop="createTime">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="操作" fixed="right" width="200">
        <template slot-scope="scope">
          <el-button
            v-if="user_upd"
            size="small"
            type="success"
            @click="formEdit(scope.row)"
            v-waves
          >编辑</el-button>
          <el-button
            v-if="user_del"
            size="small"
            type="danger"
            @click="handleDelete(scope.row)"
            v-waves
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- //  分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination
        background
        @size-change="handlePageNumChange"
        @current-change="handlePageNoChange"
        :current-page.sync="listPageParams.pageNo + 1"
        :page-sizes="[20, 40, 60, 80, 100]"
        :page-size="listPageParams.pageNum"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>

    <!-- // from dialog -->
    <el-dialog :title="dialogTitleMap[dialogStatus]" :visible.sync="dialogFormVisible" width="40%">
      <el-form :model="editForm" :rules="editFormRules" ref="editForm" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" placeholder="请输用户名"></el-input>
        </el-form-item>

        <el-form-item v-if="dialogStatus == 'create'" label="密码" prop="password">
          <el-input type="password" v-model="editForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>

        <el-form-item label="单位名称" prop="company_name">
          <el-input v-model="editForm.company_name" placeholder="请输单位名称"></el-input>
        </el-form-item>

        <el-form-item label="单位主管" prop="company_bu">
          <el-input v-model="editForm.company_bu" placeholder="请输单位主管"></el-input>
        </el-form-item>

        <el-form-item label="单位地址" prop="company_addr">
          <el-input v-model="editForm.company_addr" placeholder="请输单位地址"></el-input>
        </el-form-item>

        <el-form-item label="单位经纬度" prop="company_position">
          <el-input v-model="editForm.company_position" placeholder="请输单位经纬度"></el-input>
        </el-form-item>

        <el-form-item label="角色" prop="roleId">
          <el-select
            class="filter-item"
            style="width: 250px"
            v-model="editForm.roleId"
            placeholder="请选择角色"
          >
            <el-option
              v-for="item in editRolesOptions"
              :key="item.roleId"
              :label="item.roleDesc"
              :value="item.roleId"
              :disabled="editIsDisabled[item.statu]"
            >
              <span style="float: left">{{ item.roleDesc }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ item.roleCode }}</span>
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="单位状态" prop="company_status">
          <el-select
            class="filter-item"
            v-model="editForm.company_status"
            placeholder="请选择单位状态"
            style="width: 250px"
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
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">文件大小不超过10M</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="formCancel('editForm')">取 消</el-button>
        <el-button
          v-if="dialogStatus == 'create'"
          type="primary"
          @click="formCreate('editForm')"
        >确 定</el-button>
        <el-button v-else type="primary" @click="formUpdate('editForm')">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { fetchUserList, delByUserId, addUser, updateUser } from '@/api/user';
import { fetchRoleList } from '@/api/role';
import { mapGetters } from 'vuex';
import waves from '@/directive/waves/index.js'; // 点击按钮时候显示水波纹动画
export default {
  directives: {
    waves
  },
  data() {
    return {
      fileList: [],
      list: null,
      listLoading: false,
      listPageParams: {
        pageNo: 0,
        pageNum: 20,
        username: ''
      },
      tableKey: 0,
      total: null,
      dialogFormVisible: false, // 表单是否显示
      dialogStatus: '',
      dialogTitleMap: {
        update: '编辑用户',
        create: '添加用户'
      },
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
          { required: true, message: '请输入单位主管', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        company_name: [
          { required: true, message: '请输入单位名称', trigger: 'blur' },
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
        roleId: [{ required: true, message: '请选择角色', trigger: 'blur' }],
        company_status: [
          { required: true, message: '请选择单位状态', trigger: 'blur' }
        ]
      },
      editRolesOptions: [],
      editIsDisabled: {
        0: false,
        1: true
      },
      statusOptions: [0, 1, 2]
    };
  },
  created() {
    this.getUserList();
    this.getRoleList();
    // 设置权限，后续将采用动态方式
    this.user_upd = this.permissions['user_upd'];
    this.user_del = this.permissions['user_del'];
    this.user_add = this.permissions['user_add'];
  },
  methods: {
    getUserList() {
      this.listLoading = true;
      fetchUserList(this.listPageParams).then(res => {
        this.list = res.data.list;
        this.total = res.data.total;
        this.listLoading = false;
      });
    },
    getRoleList() {
      fetchRoleList().then(res => {
        this.editRolesOptions = res.data;
      });
    },
    handleAdd() {
      // 添加
      this.formReset();
      this.dialogStatus = 'create';
      this.dialogFormVisible = true;
    },
    handleSearch() {
      // 搜索
      this.listPageParams.pageNo = 0;
      this.getUserList();
    },
    handlePageNumChange(_pageNum) {
      // 每页显示数量变化
      this.listPageParams.pageNum = _pageNum;
      this.getUserList();
    },
    handlePageNoChange(_pageNo) {
      // 页码变化
      this.listPageParams.pageNo = _pageNo;
      this.getUserList();
    },
    handleDelete(row) {
      this.$confirm(
        '此操作将永久删除该用户(用户名:' + row.username + '), 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(() => {
          delByUserId(row.userId).then(res => {
            if (res.data) {
              this.getUserList();
              this.$notify({
                title: '成功',
                message: '删除成功',
                type: 'success',
                duration: 2000
              });
            } else {
              this.$notify({
                title: '失败',
                message: '删除失败',
                type: 'error',
                duration: 2000
              });
            }
          });
        })
        .catch(() => {
          this.$message({ type: 'info', message: '已取消删除' });
        });
    },
    formCreate(_from) {
      const f = this.$refs;
      f[_from].validate(r => {
        if (!r) return r;
        addUser(this.editForm).then(res => {
          if (!res.data) {
            this.$notify({
              title: '失败',
              message: '添加失败',
              type: 'error',
              duration: 2000
            });
          } else {
            this.dialogFormVisible = false;
            this.getUserList();
            this.$notify({
              title: '成功',
              message: '添加成功',
              type: 'success',
              duration: 2000
            });
          }
        });
        return r;
      });
    },
    formCancel(_from) {
      this.dialogFormVisible = false;
      this.$refs[_from].resetFields();
    },
    formEdit(row) {
      // 编辑
      this.formReset();
      this.dialogStatus = 'update';
      this.dialogFormVisible = true;
      this.editForm.username = row.username;
      this.editForm.password = '';
      this.editForm.company_bu = row.company_bu;
      this.editForm.company_name = row.company_name;
      this.editForm.company_position = row.company_position;
      this.editForm.company_addr = row.company_addr;
      this.editForm.company_status = row.company_status;
      this.editForm.userId = row.userId;
    },
    formUpdate(_from) {
      const f = this.$refs;
      f[_from].validate(r => {
        if (!r) return r;
        updateUser(this.editForm).then(res => {
          if (!res.data) {
            this.$notify({
              title: '失败',
              message: '修改失败',
              type: 'error',
              duration: 2000
            });
          } else {
            this.dialogFormVisible = false;
            this.getUserList();
            this.$notify({
              title: '成功',
              message: '修改成功',
              type: 'success',
              duration: 2000
            });
          }
        });
        return r;
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
  computed: {
    ...mapGetters(['permissions'])
  },
  filters: {
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
