<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="close">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="pin" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['pin']" placeholder="please enter pin"></a-input>
        </a-form-item>
        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['supplier']" placeholder="please enter supplier"></a-input>
        </a-form-item>
        <a-form-item label="contact" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['contact']" placeholder="please enter contact"></a-input>
        </a-form-item>
        <a-form-item label="tel" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['tel']" placeholder="please enter tel"></a-input>
        </a-form-item>
        <a-form-item label="email" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['email']" placeholder="please enter email"></a-input>
        </a-form-item>
        <a-form-item label="website" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['website']" placeholder="please enter website"></a-input>
        </a-form-item>
        <a-form-item label="city" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['city']" placeholder="please enter city"></a-input>
        </a-form-item>
        <a-form-item label="address" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['address']" placeholder="please enter address"></a-input>
        </a-form-item>
        <a-form-item label="zipCode" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['zipCode']" placeholder="please enter zipCode"></a-input>
        </a-form-item>
        <a-form-item label="scope" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['scope']" placeholder="please enter scope"></a-input>
        </a-form-item>

      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'


  export default {
    name: "SupplierModal",
    components: { 
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/supplier/supplier/add",
          edit: "/supplier/supplier/edit",
        }
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        console.log(this.model)
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'pin','supplier','contact','tel','email','website','city','address','zipCode','scope','createTime','updateTime'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'pin','supplier','contact','tel','email','website','city','address','zipCode','scope','createTime','updateTime'))
      },

      
    }
  }
</script>