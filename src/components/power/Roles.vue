<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type= "primary"  @click="addRoleVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <el-table :data="rolelist" border :stripe="true">
        <el-table-column type="expand"></el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" prop="roleDesc">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"  @click="editRoleDesc(scope.row.id)" >编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleDesc(scope.row.id)" >删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-search">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog title="添加角色" :visible.sync="addRoleVisible" width="50%" @close= "addRoleClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- <el-dialog title="修改角色" :visible.sync="editRoleVisible" width="50%" @close= "editRoleClosed">
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
    </el-dialog> -->

  </div>
</template>

<script>
export default {
  data () {
    return {
      rolelist: [],
      addForm: { roleName: '', roleDesc: '' },
      addRoleVisible: false,
      addFormRules: {
        roleName: [{ required: true, message: '请输角色名', trigger: 'blur' }],
        roleDesc: [{ required: false, message: '请输角色描述', trigger: 'blur' }]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')

      if (res.meta.status !== 200) {
        return this.$message.error('获取请求失败')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },

    addRoleClosed () {
    // 清空表单
      this.$refs.addFormRef.resetFields()
    },

    // 点击确定，与校验
    addRole () {
      console.log('11111')
      this.$refs.addFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.success('添加角色失败')
        }
        this.$message.success('添加角色成功')
        // 隐藏添加用户的对话框
        this.addRoleVisible = false
        // 重新获取用户数据
        this.getRolesList()
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
