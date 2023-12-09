<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="被举报用户id" prop="dealUserId">
      <el-input v-model="dataForm.dealUserId" placeholder="被举报用户id"></el-input>
    </el-form-item>
    <el-form-item label="" prop="dealVideoId">
      <el-input v-model="dataForm.dealVideoId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="类型标题，让用户选择，详情见 枚举" prop="title">
      <el-input v-model="dataForm.title" placeholder="类型标题，让用户选择，详情见 枚举"></el-input>
    </el-form-item>
    <el-form-item label="内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="内容"></el-input>
    </el-form-item>
    <el-form-item label="举报人的id" prop="userid">
      <el-input v-model="dataForm.userid" placeholder="举报人的id"></el-input>
    </el-form-item>
    <el-form-item label="举报时间" prop="createDate">
      <el-input v-model="dataForm.createDate" placeholder="举报时间"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          dealUserId: '',
          dealVideoId: '',
          title: '',
          content: '',
          userid: '',
          createDate: ''
        },
        dataRule: {
          dealUserId: [
            { required: true, message: '被举报用户id不能为空', trigger: 'blur' }
          ],
          dealVideoId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          title: [
            { required: true, message: '类型标题，让用户选择，详情见 枚举不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '内容不能为空', trigger: 'blur' }
          ],
          userid: [
            { required: true, message: '举报人的id不能为空', trigger: 'blur' }
          ],
          createDate: [
            { required: true, message: '举报时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/ware/usersreport/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.dealUserId = data.usersReport.dealUserId
                this.dataForm.dealVideoId = data.usersReport.dealVideoId
                this.dataForm.title = data.usersReport.title
                this.dataForm.content = data.usersReport.content
                this.dataForm.userid = data.usersReport.userid
                this.dataForm.createDate = data.usersReport.createDate
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/ware/usersreport/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'dealUserId': this.dataForm.dealUserId,
                'dealVideoId': this.dataForm.dealVideoId,
                'title': this.dataForm.title,
                'content': this.dataForm.content,
                'userid': this.dataForm.userid,
                'createDate': this.dataForm.createDate
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
