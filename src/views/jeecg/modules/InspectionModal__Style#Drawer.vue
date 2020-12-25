<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="ordernumber" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['ordernumber']" placeholder="please enter ordernumber"></a-input>
        </a-form-item>
        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['supplier']" placeholder="please enter supplier"></a-input>
        </a-form-item>
        <a-form-item label="inspector" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['inspector']" placeholder="please enter inspector"></a-input>
        </a-form-item>
        <a-form-item label="depositeDate" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择depositeDate" v-decorator="['depositeDate']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="comfirmDate" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择comfirmDate" v-decorator="['comfirmDate']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="inspection Date" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择inspection Date" v-decorator="['inspectionDate']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="deliveryDate" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择deliveryDate" v-decorator="['deliveryDate']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="amount" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['amount']" placeholder="please enter amount"></a-input>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">confirm</a-button>
    <a-button type="primary" @click="handleCancel">cancel</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  
  
  export default {
    name: "InspectionModal",
    components: { 
      JDate,
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
          add: "/inspection/inspection/add",
          edit: "/inspection/inspection/edit",
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
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'ordernumber','supplier','inspector','depositeDate','comfirmDate','inspectionDate','deliveryDate','amount','createTime','updateTime'))
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
        this.form.setFieldsValue(pick(row,'ordernumber','supplier','inspector','depositeDate','comfirmDate','inspectionDate','deliveryDate','amount','createTime','updateTime'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>