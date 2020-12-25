<template>
  <a-modal
    :title="title"
    :width="modalWidth"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="close"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="old password">
          <a-input type="password" placeholder="please enter old password" v-decorator="[ 'oldpassword', validatorRules.oldpassword]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="new password">
          <a-input type="password" placeholder="please enter new password" v-decorator="[ 'password', validatorRules.password]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="confirm the new password">
          <a-input type="password" @blur="handleConfirmBlur" placeholder="confirm the new password" v-decorator="[ 'confirmpassword', validatorRules.confirmpassword]"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { putAction } from '@/api/manage'

  export default {
    name: "UserPassword",
    data () {
      return {
        title:"change password",
        modalWidth:800,
        visible: false,
        confirmLoading: false,
        validatorRules:{
          oldpassword:{
            rules: [{
              required: true, message: 'please enter old password!',
            }],
          },
          password:{
            rules: [{
              required: true, message: 'please enter new password!',
            }, {
              validator: this.validateToNextPassword,
            }],
          },
          confirmpassword:{
            rules: [{
              required: true, message: 'please confirm new password!',
            }, {
              validator: this.compareToFirstPassword,
            }],
          }
        },
        confirmDirty:false,
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        form:this.$form.createForm(this),
        url: "sys/user/updatePassword",
        username:"",
      }
    },
    methods: {
      show(uname){
        if(!uname){
          this.$message.warning("no user logged in in the current system!");
          return
        }else{
          this.username = uname
          this.form.resetFields();
          this.visible = true;
        }
      },
      handleCancel () {
        this.close()
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.disableSubmit = false;
        this.selectedRole = [];
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let params = Object.assign({username:this.username},values)
            console.log("修改密码提交数据",params)
            putAction(this.url,params).then((res)=>{
              if(res.success){
                console.log(res)
                that.$message.success(res.message);
                that.close();
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
        })
      },
      validateToNextPassword  (rule, value, callback) {
        const form = this.form;
        if (value && this.confirmDirty) {
          form.validateFields(['confirm'], { force: true })
        }
        callback();
      },
      compareToFirstPassword  (rule, value, callback) {
        const form = this.form;
        if (value && value !== form.getFieldValue('password')) {
          callback('the two passwords entered are not the same！');
        } else {
          callback()
        }
      },
      handleConfirmBlur  (e) {
        const value = e.target.value
        this.confirmDirty = this.confirmDirty || !!value
      }

    }
  }
</script>

<style scoped>

</style>

