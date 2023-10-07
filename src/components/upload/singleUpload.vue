<template> 
    <div>
        <el-upload
            action="http://gulimall-1314914186.cos.ap-beijing.myqcloud.com"
            list-type="picture"
            :multiple="false"
            :show-file-list="showFileList"
            :file-list="fileList"
            :http-request="uploadImage"
            :before-upload="beforeUpload"
            :on-remove="handleRemove"
            :on-success="handleUploadSuccess"
            :on-preview="handlePreview"
        >
            <el-button
                size="small"
                type="primary"
            >点击上传</el-button>
            <div
                slot="tip"
                class="el-upload__tip"
            >只能上传jpg/png文件，且不超过10MB</div>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
            <img
                width="100%"
                :src="fileList[0].url"
                alt=""
            >
        </el-dialog>
    </div>
</template>
<script>
import { policy } from './policy'
import { getUUID } from '@/utils'

export default {
    name: 'singleUpload',
    props: {
        value: String,
    },
    computed: {
        imageUrl() {
            return this.value
        },
        imageName() {
            if (this.value != null && this.value !== '') {
                return this.value.substr(this.value.lastIndexOf('/') + 1)
            } else {
                return null
            }
        },
        fileList() {
            return [
                {
                    name: this.imageName,
                    url: this.imageUrl,
                },
            ]
        },
        showFileList: {
            get: function () {
                return (
                    this.value !== null &&
                    this.value !== '' &&
                    this.value !== undefined
                )
            },
            set: function (newValue) {},
        },
    },
    data() {
        return {
            dialogVisible: false,
            uploadUrl: [],
        }
    },
    methods: {
        emitInput(val) {
            this.$emit('input', val)
        },
        handleRemove(file, fileList) {
            this.emitInput('')
        },
        handlePreview(file) {
            this.dialogVisible = true
        },
        uploadImage: function (file) {
            // 这里的`file`参数是要上传的文件对象
            return new Promise((resolve, reject) => {
                const xhr = new XMLHttpRequest()
                // element-ui默认使用post提交，但是我们生成的url需要使用 put 提交
                // 需要指定 put 直传，以及this.uploadUrl（后端返回的url）
                xhr.open('PUT', this.uploadUrl, true)
                // 设置上传文件的类型
                xhr.setRequestHeader('Content-Type', file.file.type) // 或者 file.type
                xhr.onreadystatechange = () => {
                    if (xhr.readyState === 4) {
                        if (xhr.status === 200) {
                            resolve(xhr.response)
                            // 进入到这里就表明上传成功了
                            // 这里对后端返回的url进行截取方便存入数据库，因为前端还需要展示图片
                            // let indexOf = this.uploadUrl.indexOf('?')
                            // this.addForm.pic = this.uploadUrl.slice(0, indexOf)
                        } else {
                            reject(xhr.statusText)
                        }
                    }
                }
                xhr.onerror = () => {
                    reject(xhr.statusText)
                }
                // 发送
                xhr.send(file.file) //或者 file
            })
        },
        beforeUpload(file) {
            let _self = this
            return new Promise((resolve, reject) => {
                policy()
                    .then((response) => {
                        this.uploadUrl = response.data
                        resolve(true)
                    })
                    .catch((err) => {
                        reject(false)
                    })
            })
        },
        handleUploadSuccess(res, file) {
            console.log('上传成功...')
            this.showFileList = true
            this.fileList.pop()
            let indexOf = this.uploadUrl.indexOf('?')
            this.fileList.push({
                name: file.name,
                url: this.uploadUrl.slice(0, indexOf),
            })
            this.emitInput(this.fileList[0].url)
        },
    },
}
</script>
<style>
</style>


