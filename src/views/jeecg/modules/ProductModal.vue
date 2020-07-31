<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="sku" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['sku']" placeholder="请输入sku"></a-input>
        </a-form-item>
        <a-form-item label="project" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['project', validatorRules.project]" placeholder="请输入project"></a-input>
        </a-form-item>
        <a-form-item label="productName" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['productName']" placeholder="请输入productName"></a-input>

        </a-form-item>
        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['supplier']" placeholder="请输入supplier"></a-input>
        </a-form-item>


<!--        <a-form-item label="paramData" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="['paramData']" placeholder="请输入paramData"></a-input>-->
<!--        </a-form-item>-->


        <a-form-item
          v-for="(k, index) in array"
          :key="k"
          :label="k"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :required="false"
        >
          <a-input
            v-decorator="[
          `paramData[${k}]`,
          {
            validateTrigger: ['change', 'blur'],
            rules: [
              {
                whitespace: true,
                message: 'Please input passenger\'s name or delete this field.',
              },
            ],
          },
        ]"
            placeholder="paramData"/>
<!--            style="width: 60%; margin-right: 8px"-->
<!--          />-->
<!--          <a-icon-->
<!--            v-if="array.length > 1"-->
<!--            class="dynamic-delete-button"-->
<!--            type="minus-circle-o"-->
<!--            :disabled="array.length === 1"-->
<!--            @click="() => remove(k)"-->
<!--          />-->
        </a-form-item>


        <a-form-item label="description" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['description']" placeholder="请输入description"></a-input>
        </a-form-item>
        <a-form-item label="photo" :labelCol="labelCol" :wrapperCol="wrapperCol">


<!--      <j-image-upload isMultiple v-decorator="['photo']"></j-image-upload>-->
          <ImageUpload v-decorator="['photo']"  ref="image"> </ImageUpload>


        </a-form-item>

      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import ImageUpload from  '@/components/jeecg/ImageUpload'
  import JImageUpload from '@/components/jeecg/JImageUpload'
  import { getAction } from '@/api/manage'

  export default {
    name: "ProductModal",
    components: { 
      JImageUpload,
      ImageUpload,
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        array: [],
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
         project: {
                    rules: [
                      { validator: (rule, value, callback) => {
                           if(!value){
                                return callback("不为空")
                           }
                           value = {
                              project:value
                           }
                            getAction("/project/project/queryByName",value).then((res)=>{
                              if(res.success){
                                console.log(res.result.paramData)
                                var string = res.result.paramData
                                var array = string.split(" ")
                                this.array = array

                                return callback()
                              }else{
                                return callback(new Error(res.message));
                              }
                            })
                          }
                      },
                    ]
                  }
        },
        url: {
          add: "/product/product/add",
          edit: "/product/product/edit",
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
          this.form.setFieldsValue(pick(this.model,'sku','project','productName','supplier','paramData','description','photoString','createTime','updateTime'))

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


            values.photoString = this.$refs.image.dataI
            alert(values.photoString)
            values.paramData =JSON.stringify(values.paramData)

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
        this.form.setFieldsValue(pick(row,'sku','project','productName','supplier','paramData','description','photoString','createTime','updateTime'))
      },

      
    }
  }
</script>