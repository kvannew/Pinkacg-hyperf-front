<template>
  <a-modal
    title="用户编辑"
    :width="640"
    :visible="visible"
    :confirmLoading="loading"
    @ok="handOk"
    @cancel="() => { $emit('cancel') }"
  >
    <a-spin :spinning="loading">
      <a-form-model
        ref='ruleForm'
        :rules="rules"
        :model="model"
        v-if="model"
        v-bind="formLayout">
        <!-- 检查是否有 id 并且大于0，大于0是修改。其他是新增，新增不显示主键ID -->
        <a-form-model-item v-show="model && model.id > 0" label="主键ID">
          <a-input v-model="model.id" disabled />
        </a-form-model-item>
        <a-form-model-item
          prop='avatar'
          label="头像">
          <a-upload
            list-type="picture-card"
            class="avatar-uploader"
            :show-upload-list="false"
            :customRequest="value => getUploadAvatar(value, 'avatar')"
            :before-upload="beforeUpload"
          >
            <img style='width: 150px' v-if="model.avatar" :src="getImg(this.avatarImg)" alt="avatar" />
            <div v-else>
              <a-icon :type="avatarUpload_loading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">
                Upload
              </div>
            </div>
          </a-upload>
        </a-form-model-item>
        <a-form-model-item
          prop='background'
          label="背景">
          <a-upload
            list-type="picture-card"
            class="avatar-uploader"
            :show-upload-list="false"
            :customRequest="value => getUploadAvatar(value, 'background')"
            :before-upload="beforeUpload"
          >
            <img style='width: 200px' v-if="model.background" :src="getImg(this.backgroundImg)" alt="avatar" />
            <div v-else>
              <a-icon :type="backgroundUpload_loading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">
                Upload
              </div>
            </div>
          </a-upload>
        </a-form-model-item>
        <a-form-model-item
          prop='name'
          label="名称">
          <span slot="label">
            name&nbsp;
            <a-tooltip title="这里输入的昵称将展示给所有人！">
              <a-icon type="question-circle-o" />
            </a-tooltip>
          </span>
          <a-input v-model="model.name"/>
        </a-form-model-item>
        <a-form-model-item
          prop='username'
          label="用户名">
          <span slot="label">
            Nickname&nbsp;
            <a-tooltip title="这里输入的用户名是你的唯一uid！">
              <a-icon type="question-circle-o" />
            </a-tooltip>
          </span>
          <a-input v-model="model.username"/>
        </a-form-model-item>
        <a-form-model-item
          prop='desc'
          label="描述">
          <span slot="label">
            Nickname&nbsp;
            <a-tooltip title="这里输入描述！">
              <a-icon type="question-circle-o" />
            </a-tooltip>
          </span>
          <a-textarea v-model="model.desc"/>
        </a-form-model-item>
        <a-form-model-item
          prop='email'
          label="邮箱">
          <a-input v-model="model.email"/>
        </a-form-model-item>
        <a-form-model-item
          prop='telephone'
          label="电话">
          <a-input v-model="model.telephone" style="width: 100%" >
          </a-input>
        </a-form-model-item>
        <a-form-model-item
          prop='password'
          label="密码">
          <a-input v-model="model.password" type="password" />
        </a-form-model-item>
        <a-form-model-item
          prop='user_role'
          label="用户角色">
          <a-select v-model="model.user_role" style="width: 200px">
            <a-select-option v-for="k in roleList" :key="k.id" :value="k.id">
              {{ k.name }}
            </a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item
          prop='answertest'
          label="用户答题得分">
          <a-input-number v-model="model.answertest"/>
        </a-form-model-item>
        <a-form-model-item
          prop='credit'
          label="用户积分">
          <a-input-number v-model="model.credit"/>
        </a-form-model-item>
        <a-form-model-item
          prop='check'
          label="启用">
          <a-switch v-model="model.check" />
        </a-form-model-item>
      </a-form-model>
    </a-spin>
  </a-modal>
</template>

<script>
import pick from 'lodash.pick'
import { getRoleList } from '@/api/role'
import { getImg } from '@/utils/util'
import { uploadImgFile } from '@/api/upload'

// 表单字段
const fields = ['id', 'avatar', 'background', 'name', 'desc', 'username', 'email', 'telephone', 'answertest', 'credit', 'password', 'user_role', 'check', 'updated_at']

export default {
  props: {
    visible: {
      type: Boolean,
      required: true
    },
    loading: {
      type: Boolean,
      default: () => false
    },
    model: {
      type: Object,
      default: () => null
    }
  },
  data () {
    this.formLayout = {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      }
    }
    return {
      rules: {
        check: [{ required: true, message: '请输入状态！' }],
        answertest: [
          { required: true, message: '请输入别名' },
          { validator: this.changeKey }
        ],
        user_role: [{ required: true, message: '请输入角色！' }],
        password: [
          {
            required: true,
            message: '请输入你的密码!'
          },
          {
            validator: this.validateToNextPassword
          }
        ],
        telephone: [
          { pattern: /^1[3|4|5|7|8][0-9]\d{8}$/, message: '请输入正确的手机号' },
          { required: true, message: '请速入电话！' }
        ],
        email: [
          {
            type: 'email',
            required: true,
            message: '邮箱格式不正确'
          },
          {
            max: 50,
            message: '邮箱不得超过50字符'
          }
        ],
        desc: [{ required: true, message: '请输入描述！' }, { min: 10, message: '描述不得小于10字符' }, { max: 50, message: '描述不得超过50字符' }],
        username: [{ required: true, message: '请输入名称！' }, { min: 4, message: '用户名不得小于4字符' }, { max: 15, message: '用户名不得超过15字符' }],
        name: [{ required: true, message: '请输入名称！' }, { min: 4, message: '用户名不得小于4字符' }, { max: 15, message: '用户名不得超过15字符' }],
        avatar: [{ required: true, message: '请输入头像！' }],
        background: [{ required: true, message: '请输入背景！' }]
      },
      roleList: {},
      avatarImg: '',
      backgroundImg: '',
      confirmDirty: false,
      avatarUpload_loading: false,
      backgroundUpload_loading: false,
      form: this.$form.createForm(this),
      getImg
    }
  },
  created () {
    this.handleRole()
    // 防止表单未注册
    fields.forEach(v => this.form.getFieldDecorator(v))

    // 当 model 发生改变时，为表单设置值
    this.$watch('model', () => {
      if (!this.avatarImg && !this.backgroundImg) {
        this.avatarImg = this.model.avatar
        this.backgroundImg = this.model.background
      }
      this.model && this.form.setFieldsValue(pick(this.model, fields))
    })
  },
  methods: {
    changeKey (rule, value, callback) {
      const rgx = /\d+/
      if (rgx.test(value)) {
        callback()
      } else {
        callback(new Error('请输入数字'))
      }
    },
    getUploadAvatar (info, value) {
      const that = this
      const formData = new FormData()
      formData.append('file', info.file)
      // 开始上传
      if (value === 'avatar') {
        this.avatarUpload_loading = true
      } else if (value === 'background') {
        this.backgroundUpload_loading = true
      }
      uploadImgFile(formData).then((res) => {
        if (res.code !== 200) {
          that.$message.error(res.message)
          if (value === 'avatar') {
            this.avatarUpload_loading = false
          } else if (value === 'background') {
            this.backgroundUpload_loading = false
          }
          return []
        }
        that.$message.success(res.message)
        if (value === 'avatar') {
          that.avatarImg = res.result.link
          that.model.avatar = res.result.data
          this.avatarUpload_loading = false
        } else if (value === 'background') {
          that.backgroundImg = res.result.link
          that.model.background = res.result.data
          this.backgroundUpload_loading = false
        }
      })
    },
    handOk () {
      this.$refs.ruleForm.validate(valid => {
        if (valid) {
          this.$emit('ok')
        } else {
          this.$message.error('请输入完整数据！')
          return false
        }
      })
    },
    async handleRole () {
      const roleLists = await getRoleList()
      this.roleList = roleLists.result.data
    },
    validateToNextPassword (rule, value, callback) {
      const form = this.form
      if (value && this.confirmDirty) {
        form.validateFields(['confirm'], { force: true })
      }
      callback()
    },
    beforeUpload (file) {
      const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png'
      if (!isJpgOrPng) {
        this.$message.error('你只能上传jpep与png文件!')
      }
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('图片大于2MB!')
      }
      return isJpgOrPng && isLt2M
    }
  }
}
</script>
