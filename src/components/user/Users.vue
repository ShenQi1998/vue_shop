<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" :clearable="true" @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"  ></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <el-table :data="userlist" border stripe>
                <el-table-column label="#" type="index"> </el-table-column>
                <el-table-column label="姓名" prop="username"> </el-table-column>
                <el-table-column label="邮箱" prop="email"> </el-table-column>
                <el-table-column label="电话" prop="mobile"> </el-table-column>
                <el-table-column label="角色" prop="role_name"> </el-table-column>
                <el-table-column label="状态">
                  <template slot-scope="scope">
                    <el-switch v-model="scope.row.mg_state" @change= "userStateChanged(scope.row)">
                    </el-switch>
                  </template>
                </el-table-column>
                <el-table-column label="操作" prop="mobile" width="180px">
                   <template slot-scope="scope">
                    <!-- {{scope.row}} --->
                    <el-tooltip class="item" effect="dark" content="修改角色" placement="top" :enterable="false">
                        <el-button type="primary" icon="el-icon-edit" size='mini' @click="showEditDialog(scope.row.id)"></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="dark" content="删除角色" placement="top" :enterable="false">
                        <el-button type="danger" icon="el-icon-delete" size='mini' @click="removeUserById(scope.row.id)"></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
                        <el-button type="warning" icon="el-icon-setting" size='mini' @click="setRole(scope.row)"></el-button>
                    </el-tooltip>
                  </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>

        <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close= "addDialogClosed">
          <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="addForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input v-model="addForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="addForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
        </el-dialog>

        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close= "editDialogClosed">
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="用户名" >
              <el-input v-model="editForm.username" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop = "email">
              <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop = "mobile">
              <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
          </span>
        </el-dialog>

      <el-dialog title="分配角色" :visible.sync="setRoleDialogVisble" width="50%">
        <span>
          <p>当前的用户:{{userInfo.username}}</p>
          <p>当前的用户:{{userInfo.role_name}}</p>
          <p>分配新角色:{{userInfo.role_name}}</p>
        </span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisble = false">取 消</el-button>
          <el-button type="primary" @click = "allotRights()">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    // 校验邮箱是否合法
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法邮箱'))
    }

    // 校验手机是否合法
    var checkMobile = (rule, value, cb) => {
      const regEmail = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法手机号'))
    }

    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: { username: '', password: '', emali: '', mobile: '' },
      addFormRules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }, { min: 3, max: 10, message: '用户名在3到10个长度', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }, { min: 6, max: 15, message: '密码在6到15个长度', trigger: 'blur' }],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }, { validator: checkEmail, trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入手机', trigger: 'blur' }, { validator: checkMobile, trigger: 'blur' }]
      },
      // 控制用户修改对话框

      // 单笔查询用户
      editForm: {},
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }, { validator: checkEmail, trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入手机', trigger: 'blur' }, { validator: checkMobile, trigger: 'blur' }]
      },
      setRoleDialogVisble: false,
      userInfo: {},
      rolesList: []
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败')
      }
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    },
    handleSizeChange (newSize) {
      console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChanged (userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败！')
      }
      this.$message.success('更新用户状态成功！')
    },
    addDialogClosed () {
      // 清空表单
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed () {
      // 清空表单
      this.$refs.editFormRef.resetFields()
    },
    // 点击确定，与校验
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.success('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户数据
        this.getUserList()
      })
    },
    // 点击修改，与校验
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 201) {
          this.$message.success('修改用户信息失败')
        }
        this.$message.success('修改用户信息成功')
        // 隐藏添加用户的对话框
        this.editDialogVisible = false
        // 重新获取用户数据
        this.getUserList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    async removeUserById (id) {
    // 弹框
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    async setRole (userInfo) {
      this.userInfo = userInfo
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data

      this.setRoleDialogVisble = true
    }
  }
}
</script>

<style lang="less" scoped>
</style>
