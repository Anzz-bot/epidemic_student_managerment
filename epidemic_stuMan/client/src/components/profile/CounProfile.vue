<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>个人信息</el-breadcrumb-item>
      <el-breadcrumb-item>查看</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 展示个人资料区域 -->
    <el-card style="width: 800px">
      <template #header>
        <div class="clearfix">
          <span style="font-weight: bold; font-size: 18px">个人资料</span>
          <el-button style="float: right; padding: 3px 0; font-size: 18px" type="text" @click="modifyDialogVisible = true">修改</el-button>
        </div>
      </template>
      <el-row>
        <el-col :span="12">
          <img src="../../assets/mao.jpeg">
        </el-col>
        <el-col :span="12">
          <div v-for="(item,i) in labelList" :key="i">
            <div style="font-size: 18px;margin-top: 20px" class="text item">{{item}}: {{personInfo[propList[i]]}}</div>
          </div>
        </el-col>
      </el-row>
    </el-card>
    <!-- 修改用户的对话框   -->
    <el-dialog title="修改个人资料" v-model="modifyDialogVisible" width="40%" @close="showProfile">
      <el-form v-for="(item,i) in labelList" :key="i" :model="personInfo"
               :rules="modifyFormRules" ref="modifyFormRef" label-width="80px">
        <el-form-item :label="item" :prop="propList[i]">
          <!-- 工号和身份不可修改 -->
          <el-input v-if="item === '工号' || item === '身份'" v-model="personInfo[propList[i]]" disabled></el-input>
          <el-input v-else v-model="personInfo[propList[i]]"></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="modifyDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="modifyProfile">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      id: '', // 当前用户的id
      identity: '', // 当前用户的身份
      // 存放个人资料
      personInfo: { id: '', name: '', sex: '', identity: '', college: '', tel: '', pwd: '' },
      modifyDialogVisible: false, // 控制修改个人资料对话框是否可见
      // 个人资料中的关键字
      labelList: ['工号', '姓名', '性别', '身份', '学院', '联系方式', '账号密码'],
      propList: ['id', 'name', 'sex', 'identity', 'college', 'tel', 'pwd'],
      // identity和身份的映射
      identity_reflect: { 0: '管理员', 1: '学生', 2: '辅导员' },
      // 修改表单的验证规则对象(主要验证是否为空),这里的key顺序和上面propList一致
      modifyFormRules: {
        id: [{ required: true, message: '不能为空', trigger: 'blur' }],
        name: [{ required: true, message: '不能为空', trigger: 'blur' }],
        sex: [{ required: true, message: '不能为空', trigger: 'blur' }],
        identity: [{ required: true, message: '不能为空', trigger: 'blur' }],
        college: [{ required: true, message: '不能为空', trigger: 'blur' }],
        tel: [{ required: true, message: '不能为空', trigger: 'blur' }],
        pwd: [{ required: true, message: '不能为空', trigger: 'blur' }]
      }
    }
  },
  created () {
    this.id = window.sessionStorage.getItem('id')
    this.showProfile()
  },
  methods: {
    // 获取个人信息并展示
    async showProfile () {
      try {
        const { data: res } = await this.$http.get('/pi/coun/' + this.id)
        res.data.identity = this.identity_reflect[res.data.identity]
        this.personInfo = res.data
      } catch (err) {
        return this.$message.error('获取个人资料失败！')
      }
    },
    // 修改个人信息的对话框
    modifyProfile () {
      this.$refs.modifyFormRef[0].validate(async valid => {
        if (!valid) return
        try {
          await this.$http.put('/pi/coun/' + this.id, {
            name: this.personInfo.name,
            sex: this.personInfo.sex,
            college: this.personInfo.college,
            tel: this.personInfo.tel,
            pwd: this.personInfo.pwd
          })
          this.$message.success('更新个人资料成功！')
          this.modifyDialogVisible = false
          await this.showProfile()
        } catch (err) {
          return this.$message.error('更新个人资料失败！')
        }
      })
    }
  }
}
</script>

<style scoped>
</style>
