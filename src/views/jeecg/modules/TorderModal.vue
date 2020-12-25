<template>
<div>
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

        <a-form-item label="ordernumber" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['ordernumber']" placeholder="please enter ordernumber"></a-input>
        </a-form-item>
<!--        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="['supplier', validatorRules.supplier]" placeholder="please enter supplier" ></a-input>-->
<!--          <ListItem v-decorator="['supplier', validatorRules.supplier]" placeholder="please enter supplier" ></ListItem>-->

<!--          <a-button style="margin-right: 8px;" @click="()=>this.$refs.modalForm.visible=true">enter supplier detail</a-button>-->
<!--          <supplier-modal ref="modalForm" @ok="modalFormOk"></supplier-modal>-->
<!--        </a-form-item>-->
        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol"  has-feedback  @click.native="clickS()">
          <a-select
            v-decorator="['supplier', validatorRules.supplier]"
            placeholder="Please select a supplier"
            style="width: 500px"
          >
            <a-select-option :value="spr.supplier" v-for="spr in suppliers">
              {{ spr.supplier }}
            </a-select-option>
          </a-select>
          <a-button style="margin-right: 8px;" @click="()=>this.$refs.modalForm.visible=true">enter supplier detail</a-button>
        </a-form-item>

        <supplier-modal ref="modalForm" @ok="modalFormOk"></supplier-modal>


        <a-form-item label="client" :labelCol="labelCol" :wrapperCol="wrapperCol"  @click.native="clickC()" has-feedback>
          <a-select
            v-decorator="['client', validatorRules.client]"
            placeholder="Please select a client"
            style="width: 500px"

          >
            <a-select-option :value="cli.client" v-for="cli in clients">
              {{ cli.client }}
            </a-select-option>
          </a-select>
          <a-button style="margin-right: 8px;" @click="()=>this.$refs.cmodalForm.visible=true">enter client detail</a-button>
          <client-modal ref="cmodalForm" @ok="modalFormOk"></client-modal>
        </a-form-item>





        <a-form-item label="chop" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['chop']" placeholder="please enter chop"></a-input>
        </a-form-item>
        <a-form-item label="season" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-model="objseason" v-decorator="['season']" placeholder="please enter season"></a-input>
        </a-form-item>

<!--        <a-form-item label="season" :labelCol="labelCol" :wrapperCol="wrapperCol"  has-feedback>-->
<!--          <a-select-->
<!--            v-decorator="['season', validatorRules.season]"-->
<!--            placeholder="Please select a season"-->
<!--            style="width: 500px"-->
<!--          >-->
<!--            <a-select-option :value="ses" v-for="ses in seasons">-->
<!--              {{ ses }}-->
<!--            </a-select-option>-->
<!--          </a-select>-->
<!--        </a-form-item>-->







        <a-form-item label="product" :labelCol="labelCol" :wrapperCol="wrapperCol" v-model="sku">
          <a-input v-decorator="['product', validatorRules.product]" placeholder="please enter product"></a-input>
          <a-button style="margin-right: 8px;" @click="()=>this.$refs.modalFormp.visible=true">enter product detail</a-button>
          <product-modal ref="modalFormp" @ok="modalFormOk"></product-modal>
        </a-form-item>
        <a-form-item label="incoterm" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['incoterm']" placeholder="please enter incoterm"></a-input>
        </a-form-item>
        <a-form-item label="delivery" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['delivery']" placeholder="please enter delivery"></a-input>
        </a-form-item>
        <a-form-item label="priceUsd" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['priceUsd']" placeholder="please enter price"></a-input>
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
  </j-modal>


   </div>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import { getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'
  import SupplierModal from './SupplierModal'
  import ClientModal from './ClientModal'
  import ProductModal from './ProductModal'
  import ListItem from '@/components/jeecg/modal/ListItem'

  export default {
    name: "TorderModal",
    components: {
      JDate,
      SupplierModal,
      ClientModal,
      ProductModal,
      ListItem,
    },
    data () {
      return {
        sku:"",
        seasons:{},
        objseason: "",
        suppliers: {},
        clients:{},
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
              { validator: (rule, value, callback) => {
                   if(!value){
                        return callback("not null")
                   }else {
                     return callback()
                   }
                   // value = {
                   //    supplier:value
                   // }
                   // getAction("/supplier/supplier/queryByName",value).then((res)=>{
                   //    if(res.success){
                   //        return callback()
                   //    }else{
                   //        return callback(new Error(res.message));
                   //    }
                   //  })
                  }
              },
            ]
          },
          client: {
             rules: [
                         { validator: (rule, value, callback) => {
                              if(!value){
                                   return callback("not null")
                              }else {
                                return callback()
                              }
                              // value = {
                              //    client:value
                              // }
                              // getAction("/client/client/queryByName",value).then((res)=>{
                              //    if(res.success){
                              //        return callback()
                              //    }else{
                              //        return callback(new Error(res.message));
                              //    }
                              //  })
                             }
                         },
                       ]
          },
          product: {
            rules: [
                     { validator: (rule, value, callback) => {
                              if(!value){
                                return callback("not null")
                              }else {
                                return callback()
                              }
                             // let value1 = {
                             //   product:{"season":this.objseason,"product":this.sku}
                             //  }
                             //
                             //  getAction("/product/product/queryByName",value1).then((res)=>{
                             //    if(res.success){
                             //      return callback()
                             //    }else{
                             //      return callback(new Error(res.message));
                             //    }
                             //  })
                          }
                     },
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
      getAction("/supplier/supplier/listAll").then((res)=>{
        if(res.success){


          this.suppliers= res.result

        }else{
          alert("supplier false")
        }
      }),
      getAction("/client/client/listAll").then((res)=>{
        if(res.success){
          this.clients = res.result

        }else{
          alert("clent false")
        }
      }),
      getAction("/torder/torder/season").then((res)=>{
        if(res.success){
          this.seasons = res.result
          console.log(res)
        }else{
          alert("season false")
        }
      })
    },


    methods: {
      // clickS(){
      //   getAction("/supplier/supplier/listAll").then((res)=>{
      //     if(res.success){
      //       this.suppliers = res.result
      //     }else{
      //       alert("supplier false")
      //     }
      //   })
      // },
      // clickC(){
      //   getAction("/client/client/listAll").then((res)=>{
      //     if(res.success){
      //       this.clients = res.result
      //
      //     }else{
      //       alert("clent false")
      //     }
      //   })
      // },

      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.sku = record.product
          this.objseason = record.season
          this.form.setFieldsValue(pick(this.model,'ordernumber','supplier','client','chop','season','product','incoterm','delivery','priceUsd','quantity','amountRmb','amountUsd','depositRmb','depositUsd','depositPay','balanceRmb','balanceUsd','balancePay','deliveryDate','deliverySituation','etd','delay','eta','status','statusdate','created','updated'))

        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        var d = new Date()
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
            alert("表单提交数据")
            var d1 = new Date()
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
            var d2 = new Date()
            console.log(d)
            console.log(d1)
            console.log(d2)
          }

        })

      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'ordernumber','supplier','client','chop','season','product','incoterm','delivery','priceUsd','quantity','amountRmb','amountUsd','depositRmb','depositUsd','depositPay','balanceRmb','balanceUsd','balancePay','deliveryDate','deliverySituation','etd','delay','eta','status','statusdate','created','updated'))
      },

      
    }
  }
</script>