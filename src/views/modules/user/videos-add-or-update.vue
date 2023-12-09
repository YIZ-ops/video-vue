<template>
    <el-dialog
        :title="!dataForm.id ? '新增' : '修改'"
        :close-on-click-modal="false"
        :visible.sync="visible"
    >
        <el-form
            :model="dataForm"
            :rules="dataRule"
            ref="dataForm"
            @keyup.enter.native="dataFormSubmit()"
            label-width="100px"
        >
            <el-form-item
                label="视频描述"
                prop="videoDesc"
            >
                <el-input
                    v-model="dataForm.videoDesc"
                    placeholder="视频描述"
                ></el-input>
            </el-form-item>
            <el-form-item
                label="视频文件"
                prop="videoPath"
            >
                <el-upload
                    :action="uploadVideoUrl"
                    :on-success="handleVideoUploadSuccess"
                    limit="1"
                    accept=".mp4"
                >
                    <el-button
                        slot="trigger"
                        size="small"
                        type="primary"
                    >选择文件</el-button>
                </el-upload>
                <div v-if="dataForm.videoPath">{{ dataForm.videoPath }}</div>
            </el-form-item>
            <el-form-item
                label="视频封面图"
                prop="coverPath"
            >
                <el-upload
                    :action="uploadCoverUrl"
                    :on-success="handleCoverUploadSuccess"
                    limit="1"
                    accept=".jpg,.jpeg,.png"
                >
                    <el-button
                        slot="trigger"
                        size="small"
                        type="primary"
                    >选择图片</el-button>
                </el-upload>
                <div v-if="dataForm.coverPath">{{ dataForm.coverPath }}</div>
            </el-form-item>
            <el-form-item
                label="视频状态"
                prop="status"
            >
                <el-radio-group v-model="dataForm.status">
                    <el-radio label="0">发布成功</el-radio>
                    <el-radio label="1">禁止播放</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item
                label="视频类型"
                prop="kind"
            >
                <el-radio-group v-model="dataForm.kind">
                    <el-radio label="0">实用</el-radio>
                    <el-radio label="1">享乐</el-radio>
                </el-radio-group>
            </el-form-item>
        </el-form>
        <span
            slot="footer"
            class="dialog-footer"
        >
            <el-button @click="visible = false">取消</el-button>
            <el-button
                type="primary"
                @click="dataFormSubmit()"
            >确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
export default {
    data() {
        return {
            uploadVideoUrl: '',
            visible: false,
            dataForm: {
                id: 0,
                videoDesc: '',
                videoPath: '',
                coverPath: '',
                likeCounts: 0,
                status: 0,
                createTime: '',
                kind: 0,
            },
            dataRule: {
                videoDesc: [
                    {
                        required: true,
                        message: '视频描述不能为空',
                        trigger: 'blur',
                    },
                ],
                videoPath: [
                    {
                        required: true,
                        message: '视频存放的路径不能为空',
                        trigger: 'blur',
                    },
                ],
                coverPath: [
                    {
                        required: true,
                        message: '视频封面图不能为空',
                        trigger: 'blur',
                    },
                ],
                status: [
                    {
                        required: true,
                        message: '视频状态：1、发布成功2、禁止播放',
                        trigger: 'blur',
                    },
                ],
                kind: [
                    {
                        required: true,
                        message: '0-实用 1-享乐不能为空',
                        trigger: 'blur',
                    },
                ],
            },
        }
    },
    mounted() {
        this.uploadVideoUrl = this.$http.adornUrl('/ware/videos/upload-video')
        this.uploadCoverUrl = this.$http.adornUrl('/ware/videos/upload-cover')
    },
    methods: {
        init(id) {
            this.dataForm.id = id || 0
            this.visible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].resetFields()
                if (this.dataForm.id) {
                    this.$http({
                        url: this.$http.adornUrl(
                            `/ware/videos/info/${this.dataForm.id}`
                        ),
                        method: 'get',
                        params: this.$http.adornParams(),
                    }).then(({ data }) => {
                        if (data && data.code === 0) {
                            this.dataForm.userId = data.videos.userId
                            this.dataForm.audioId = data.videos.audioId
                            this.dataForm.videoDesc = data.videos.videoDesc
                            this.dataForm.videoPath = data.videos.videoPath
                            this.dataForm.videoSeconds =
                                data.videos.videoSeconds
                            this.dataForm.videoWidth = data.videos.videoWidth
                            this.dataForm.videoHeight = data.videos.videoHeight
                            this.dataForm.coverPath = data.videos.coverPath
                            this.dataForm.likeCounts = data.videos.likeCounts
                            this.dataForm.status = data.videos.status.toString()
                            this.dataForm.createTime = data.videos.createTime
                            this.dataForm.kind = data.videos.kind.toString()
                        }
                    })
                }
            })
        },
        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(
                            `/ware/videos/${
                                !this.dataForm.id ? 'save' : 'update'
                            }`
                        ),
                        method: 'post',
                        data: this.$http.adornData({
                            id: this.dataForm.id || undefined,
                            videoDesc: this.dataForm.videoDesc,
                            videoPath: this.dataForm.videoPath,
                            coverPath: this.dataForm.coverPath,
                            likeCounts: this.dataForm.likeCounts,
                            status: this.dataForm.status,
                            kind: this.dataForm.kind,
                        }),
                    }).then(({ data }) => {
                        if (data && data.code === 0) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                },
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                }
            })
        },
        handleVideoUploadSuccess(res) {
            this.dataForm.videoPath = res.finalVideoPath
        },
        handleCoverUploadSuccess(res) {
            this.dataForm.coverPath = res.finalCoverPath
        },
        resetFields() {
            this.dataForm.videoPath = ''
            this.dataForm.coverPath = ''
        },
    },
}
</script>
