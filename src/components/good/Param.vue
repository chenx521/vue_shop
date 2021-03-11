<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-alert
        title="注意：只允许为第三级分类设置相关参数!"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <el-row class="span-title">
        <el-col>
          <span> 选择商品分类： </span>
          <el-cascader
            :options="parentCateList"
            @change="handleChange"
            :props="cateProps"
            v-model="pIdValues"
            clearable
            ref="cascaderHandle"
          ></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isTabsActive"
            @click="addParamMany"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyDataList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="ediParamDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="warning"
                  icon="el-icon-delete"
                  @click="deleteParam(scope.row)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            :disabled="isTabsActive"
            @click="addParamMany"
            >添加属性</el-button
          >
          <el-table :data="onlyDataList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="ediParamDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="warning"
                  icon="el-icon-delete"
                  @click="deleteParam(scope.row)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
      <el-dialog
        :title="dialogTitle"
        :visible.sync="addManyDialogVisible"
        width="50%"
        @close="closeAddParamDialog"
      >
        <el-form
          :model="addParamForm"
          :rules="addParamRules"
          ref="addParamFormRef"
          label-width="100px"
        >
          <el-form-item :label="lableInfo" prop="attr_name">
            <el-input v-model="addParamForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addManyDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addManyOrOnly">确 定</el-button>
        </span>
      </el-dialog>

      <el-dialog
        :title="'修改' + lableInfo"
        :visible.sync="editManyDialogVisible"
        width="50%"
        @close="closeEditParamDialog"
      >
        <el-form
          :model="editParamForm"
          :rules="editParamRules"
          ref="editParamFormRef"
          label-width="100px"
        >
          <el-form-item :label="lableInfo" prop="attr_name">
            <el-input v-model="editParamForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editManyDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editManyOrOnly">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      parentCateList: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
        expandTrigger: 'hover'
      },
      pIdValues: [],
      activeName: 'many',
      manyDataList: [],
      onlyDataList: [],
      addManyDialogVisible: false,
      addParamForm: {},
      addParamRules: {
        attr_name: [
          { required: true, message: '请输入动态参数名称', trigger: 'blur' }
        ]
      },
      dialogTitle: '添加动态参数',
      lableInfo: '动态参数',
      editManyDialogVisible: false,
      editParamForm: {
        attr_name: '',
        attr_id: ''
      },
      editParamRules: {
        attr_name: [
          { required: true, message: '请输入动态参数名称', trigger: 'blur' }
        ]
      },
      inputVisible: false,
      inputValue: ''
    }
  },
  created() {
    this.getParentCateList()
  },
  methods: {
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 3 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      this.parentCateList = res.data
    },
    async handleChange() {
      if (this.pIdValues.length !== 3) {
        this.pIdValues = []
        this.manyDataList = []
        this.onlyDataList = []
        return
      }
      /* 关闭下拉框 */
      this.$refs.cascaderHandle.dropDownVisible = false
      this.getParamsData()
    },
    handleClick(tab, event) {
      console.log(this.activeName)
      if (this.activeName === 'many') {
        this.dialogTitle = '添加动态参数'
        this.lableInfo = '动态参数'
      } else {
        this.dialogTitle = '添加静态属性'
        this.lableInfo = '静态属性'
      }
      this.getParamsData()
    },
    async getParamsData() {
      /* 获取tab数据 */
      const { data: res } = await this.$http.get(
        `categories/${this.getCateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表数据失败！')
      }
      res.data.forEach((item) => {
        console.log(item.attr_vals)
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyDataList = res.data
      } else {
        this.onlyDataList = res.data
      }
    },
    addParamMany() {
      this.addManyDialogVisible = true
    },
    closeAddParamDialog() {
      this.$refs.addParamFormRef.resetFields()
    },
    ediParamDialog(row) {
      this.editManyDialogVisible = true
      this.editParamForm.attr_id = row.attr_id
      this.editParamForm.attr_name = row.attr_name
    },
    /* 添加动态参数或者静态属性 */
    addManyOrOnly() {
      /* 记得进行表单的预验证 */
      this.$refs.addParamFormRef.validate(async (valid) => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          `categories/${this.getCateId}/attributes`,
          {
            attr_name: this.addParamForm.attr_name,
            attr_sel: this.activeName /* ,
            attr_vals:  */
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加信息失败！')
        }
        this.getParamsData()
        this.$message.success('添加信息成功！')
        this.addManyDialogVisible = false
        console.log(res)
      })
    },
    editManyOrOnly() {
      this.$refs.editParamFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.getCateId}/attributes/${this.editParamForm.attr_id}`,
          {
            attr_name: this.editParamForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改信息失败')
        }
        this.getParamsData()
        this.$message.success('修改信息成功！')
        this.editManyDialogVisible = false
      })
    },
    closeEditParamDialog() {
      this.$refs.editParamFormRef.resetFields()
    },
    /* 删除参数 */
    async deleteParam(row) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => {
        return err
      })
      if (confirmResult === 'confirm') {
        const { data: res } = await this.$http.delete(
          `categories/${this.getCateId}/attributes/${row.attr_id}`
        )
        if (res.meta.status !== 200) {
          return this.$message.error('删除失败')
        }
        this.getParamsData()
        return this.$message.success('删除成功!')
      }
      return this.$message.info('取消删除!')
    },
    showInput(row) {
      row.inputVisible = true
      /* nextTick 当页面的元素渲染之后才会执行回到函数 否则报错 */
      this.$nextTick((_) => {
        /* 让文本框自动获得焦点 */
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleInputConfirm(row) {
      if (row.inputValue.trim().length !== 0) {
        row.attr_vals.push(row.inputValue)
        row.inputVisible = false
        row.inputValue = ''
        this.saveEditParam(row)
        /*         this.getParamsData()
         */
      }
    },
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveEditParam(row)
    },
    async saveEditParam(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.getCateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(',')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('操作失败！')
      }
      this.$message.success('操作成功！')
    }
  },
  /* 计算属性 控制按钮是否可用 */
  computed: {
    isTabsActive() {
      if (this.pIdValues.length !== 3) {
        return true
      }
      return false
    },
    getCateId() {
      if (this.pIdValues.length !== 3) {
        return null
      }
      return this.pIdValues[this.pIdValues.length - 1]
    }
  }
}
</script>
<style lang="less" scoped>
/* 设置下拉框的top距离 不适用于全局 */
@import '../../assets/css/param_dropdown.css';
.span-title {
  margin: 10px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 90px;
}
</style>
