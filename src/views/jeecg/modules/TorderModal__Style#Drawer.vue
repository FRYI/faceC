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
          <a-input v-decorator="['supplier', validatorRules.supplier]" placeholder="please enter supplier"></a-input>
        </a-form-item>
        <a-form-item label="client" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['client', validatorRules.client]" placeholder="please enter client"></a-input>
        </a-form-item>
        <a-form-item label="chop" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['chop']" placeholder="please enter chop"></a-input>
        </a-form-item>
        <a-form-item label="product" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['product', validatorRules.product]" placeholder="please enter product"></a-input>
        </a-form-item>
        <a-form-item label="incoterm" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['incoterm']" placeholder="please enter incoterm"></a-input>
        </a-form-item>
        <a-form-item label="delivery" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['delivery']" placeholder="please enter delivery"></a-input>
        </a-form-item>
        <a-form-item label="price" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['price']" placeholder="please enter price"></a-input>
        </a-form-item>
        <a-form-item label="quantity" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['quantity']" placeholder="please enter quantity"></a-input>
        </a-form-item>
        <a-form-item label="amountRmb" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['amountRmb']" placeholder="please enter amountRmb"></a-input>
        </a-form-item>
        <a-form-item label="amountUsd" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['amountUsd']" placeholder="please enter amountUsd"></a-input>
        </a-form-item>
        <a-form-item label="depositRmb" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['depositRmb']" placeholder="please enter depositRmb"></a-input>
        </a-form-item>
        <a-form-item label="depositUsd" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['depositUsd']" placeholder="please enter depositUsd"></a-input>
        </a-form-item>
        <a-form-item label="depositPay" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['depositPay']" placeholder="please enter depositPay"></a-input>
        </a-form-item>
        <a-form-item label="balanceRmb" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['balanceRmb']" placeholder="please enter balanceRmb"></a-input>
        </a-form-item>
        <a-form-item label="balanceUsd" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['balanceUsd']" placeholder="please enter balanceUsd"></a-input>
        </a-form-item>
        <a-form-item label="balancePay" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['balancePay']" placeholder="please enter balancePay"></a-input>
        </a-form-item>
        <a-form-item label="deliveryDate" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择deliveryDate" v-decorator="['deliveryDate']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="deliverySituation" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['deliverySituation']" placeholder="please enter deliverySituation"></a-input>
        </a-form-item>
        <a-form-item label="etd" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择etd" v-decorator="['etd']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="delay" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['delay']" placeholder="please enter delay"></a-input>
        </a-form-item>
        <a-form-item label="eta" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择eta" v-decorator="['eta']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="status" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['status']" placeholder="please enter status"></a-input>
        </a-form-item>
        <a-form-item label="statusdate" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择statusdate" v-decorator="['statusdate']" :trigger-change="true" style="width: 100%"/>
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
    name: "TorderModal",
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
          supplier: {
            rules: [
              { validator: (rule, value, callback) => validateDuplicateValue('torder', 'supplier', value, this.model.id, callback)},
            ]
          },
          client: {
            rules: [
              { validator: (rule, value, callback) => validateDuplicateValue('torder', 'client', value, this.model.id, callback)},
            ]
          },
          product: {
            rules: [
              { validator: (rule, value, callback) => validateDuplicateValue('torder', 'product', value, this.model.id, callback)},
            ]
          },
        },
        url: {
          add: "/torder/torder/add",
          edit: "/torder/torder/edit",
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
          this.form.setFieldsValue(pick(this.model,'ordernumber','supplier','client','chop','product','incoterm','delivery','price','quantity','amountRmb','amountUsd','depositRmb','depositUsd','depositPay','balanceRmb','balanceUsd','balancePay','deliveryDate','deliverySituation','etd','delay','eta','status','statusdate','createTime','updateTime'))
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
        this.form.setFieldsValue(pick(row,'ordernumber','supplier','client','chop','product','incoterm','delivery','price','quantity','amountRmb','amountUsd','depositRmb','depositUsd','depositPay','balanceRmb','balanceUsd','balancePay','deliveryDate','deliverySituation','etd','delay','eta','status','statusdate','createTime','updateTime'))
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