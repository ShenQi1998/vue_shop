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
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag :closable="true" @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" :closable="true" @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" :closable="true" @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" prop="roleDesc">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"  @click="editRoleDesc(scope.row.id)" >编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)" >删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-search" @click="showSetRightDialog(scope.row)">分配权限</el-button>
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

    <el-dialog title="修改用户" :visible.sync="editRoleVisible" width="50%" @close= "editRoleClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" :default-expand-all="true" :default-checked-keys="defkeys" ref = "treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click = "allotRights()">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      rolelist: [],
      addForm: { roleName: '', roleDesc: '' },
      editForm: {},
      rightslist: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defkeys: [],
      addRoleVisible: false,
      editRoleVisible: false,
      setRightDialogVisible: false,
      addFormRules: {
        roleName: [{ required: true, message: '请输角色名', trigger: 'blur' }],
        roleDesc: [{ required: false, message: '请输角色描述', trigger: 'blur' }]
      },
      editFormRules: {
        roleName: [{ required: true, message: '请输角色名', trigger: 'blur' }],
        roleDesc: [{ required: false, message: '请输角色描述', trigger: 'blur' }]
      },
      roleId: ''
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

    editRoleClosed () {
    // 清空表单
      this.$refs.editFormRef.resetFields()
    },

    // 添加
    addRole () {
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
    },

    // 修改
    editRoleInfo () {
      this.$refs.editFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })

        if (res.meta.status !== 201) {
          this.$message.success('修改用户信息失败')
        }
        this.$message.success('修改用户信息成功')
        // 隐藏添加用户的对话框
        this.editDialogVisible = false
        // 重新获取用户数据
        this.getRolesList()
      })
    },

    // 单笔查询
    async editRoleDesc (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      this.editRoleVisible = true
    },

    async removeRoleById (id) {
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getRolesList()
    },

    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },

    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      this.rightslist = res.data
      this.defkeys = []
      this.getLeafKeys(role, this.defkeys)
      // console.log(this.rightslist)
      this.setRightDialogVisible = true
    },

    getLeafKeys (node, array) {
      if (!node.children) {
        return array.push(node.id)
      }

      node.children.forEach(item => this.getLeafKeys(item, array))
    },

    async allotRights (role) {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      console.log(keys)

      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
