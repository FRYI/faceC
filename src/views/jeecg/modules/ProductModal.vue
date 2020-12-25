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

        <a-form-item label="season" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-model="objseason" v-decorator="['season']" placeholder="please enter season"></a-input>
        </a-form-item>
<!--        <a-form-item label="season" :labelCol="labelCol" :wrapperCol="wrapperCol"  has-feedback>-->
<!--          <a-select-->
<!--            v-decorator="['season']"-->
<!--            placeholder="Please select a season"-->
<!--            style="width: 500px"-->
<!--          >-->
<!--            <a-select-option :value="ses" v-for="ses in seasons">-->
<!--              {{ ses }}-->
<!--            </a-select-option>-->
<!--          </a-select>-->
<!--        </a-form-item>-->
        <a-form-item label="sku" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['sku']" placeholder="please enter sku"></a-input>
        </a-form-item>
<!--        <a-form-item label="project" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="['project', validatorRules.project]" placeholder="please enter project"></a-input>-->
<!--        </a-form-item>-->

        <a-form-item label="project" :labelCol="labelCol" :wrapperCol="wrapperCol"  @click.native="clickP()" has-feedback>
          <a-select
            v-decorator="['project', validatorRules.project]"
            placeholder="Please select a project"
            style="width: 500px"
          >
            <a-select-option :value="prt.projectName" v-for="prt in projects">
              {{ prt.projectName }}
            </a-select-option>
          </a-select>
        </a-form-item>




        <a-form-item label="productName" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['productName']" placeholder="please enter productName"></a-input>
        </a-form-item>

<!--        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="['supplier']" placeholder="supplier"></a-input>-->
<!--        </a-form-item>-->

        <a-form-item label="supplier" :labelCol="labelCol" :wrapperCol="wrapperCol"  @click.native="clickS()" has-feedback>
          <a-select
            v-decorator="['supplier']"
            placeholder="Please select a supplier"
            style="width: 500px"
          >
            <a-select-option :value="spr.supplier" v-for="spr in suppliers">
              {{ spr.supplier }}
            </a-select-option>
          </a-select>
        </a-form-item>


        <a-form-item
          v-for="(value,k,index) in array"
          :key="k"
          :label="k"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :required="false"
        >
          <a-input
            v-decorator="[
          `paramData[${k}]`,
          // '${k}',
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
        </a-form-item>


        <a-form-item label="description" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['description']" placeholder="please enter description"></a-input>
        </a-form-item>
        <a-form-item label="photo" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <ImageIterm v-decorator="['photo']"  ref="image" @imagedatapost="imgget" > </ImageIterm>
          <img :src="imgsrc" v-show="imgsrc"/>
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
  import ImageIterm from '@/components/jeecg/modal/ImageIterm'
  import { getAction } from '@/api/manage'

  export default {
    name: "ProductModal",
    props:{
      "immm":""
    },
    components: { 
      JImageUpload,
      ImageUpload,
      ImageIterm,
    },
    data () {
      return {
        vi:"",
        season:{},
        suppliers:{},
        projects:{},
        imgsrc: "",
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        array: [],
        img:"",
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
                                var obj ={}
                                for (var key in array) {
                                  obj[array[key]] = "";
                                }

                                this.array = obj
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
    beforeCreate() {
      let value = {
        project:record.project
      }
      getAction("/project/project/queryByName",value).then((res)=>{
        if(res.success){
          console.log(res.result.paramData)
          var string = res.result.paramData
          var array = string.split(" ")
          var obj ={}
          for (var key in array) {
            obj[array[key]] = "";
            this.getFieldDecorator("paramData["+key+"]", { initialValue: [], preserve: false })
          }
          this.array = obj


        }else{
          console.log("project false")
        }
      })
    },
    created () {
      getAction("/supplier/supplier/listAll").then((res)=>{
        if(res.success){
          this.suppliers = res.result
        }else{
          alert("supplier false")
        }
      }),
        getAction("/project/project/listAll").then((res)=>{
          if(res.success){
            this.projects = res.result
            console.log(res)
          }else{
            alert("project false")
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
    mounted() {


    },
    methods: {

      clickS(){
        getAction("/supplier/supplier/listAll").then((res)=>{
          if(res.success){
            this.suppliers = res.result
          }else{
            alert("supplier false")
          }
        })
      },
      clickP(){
        getAction("/project/project/listAll").then((res)=>{
          if(res.success){
            this.projects = res.result
            console.log(res)
          }else{
            alert("project false")
          }
        })
      },
      add () {
        this.edit({});
      },
      edit (record) {

        console.log(record)
        this.form.resetFields();

        this.model = Object.assign({}, record);

        this.visible = true;
        this.$nextTick(() => {
          let value = {
            project:record.project
          }
          getAction("/project/project/queryByName",value).then((res)=>{
            if(res.success){
              console.log(res.result.paramData)
              var string = res.result.paramData
              var array = string.split(" ")
              var obj ={}
              for (var key in array) {
                obj[array[key]] = "";
              }
              this.array = obj
            }else{
              console.log("project false")
            }
          })

          console.log("modelData"+this.model)
          console.log(this.model)
          this.form.setFieldsValue(pick(this.model,'season','sku','project','productName','supplier','description','photoString','createTime','updateTime'))
          this.imgsrc = record.photoString;

          var json2 = JSON.parse(record.paramData)
          Object.keys(json2).map((obj, idx) => (
            setTimeout(()=>{

              var cs = "paramData["+obj+"]".toString()

              // this.form.setFieldsValue({ ["paramData[color]"]:json2[obj]});
              this.form.setFieldsValue({ [cs]:json2[obj]});
            },1000)
          ))



        })
      },
      close () {
        console.log(this.$refs)
        this.$emit('close');
        this.visible = false;
        this.imgsrc = ""
        this.img=""
        this.$refs.image.imgData = ""
        this.$refs.image.clearImg();
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
            if(this.img&&!this.imgsrc){
              values.photoString=this.img
            }else if(this.imgsrc&&!this.img){
              values.photoString = this.imgsrc
            }else if ((this.imgsrc && this.img)){
              values.photoString = this.img
            }

            values.paramData =JSON.stringify(values.paramData)


            console.log(values.paramData)

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
        this.form.setFieldsValue(pick(row,'season','sku','project','productName','supplier','paramData','description','photoString','createTime','updateTime'))
      },
      imgget(data){
        this.img = data;
      },
    }
  }
</script>
<style>
  #image{
    display: inline-block;
  }

  img:last-child{
    margin-top: -90px;
    margin-left: 10px;
    width: 100px;
    height: 100px;
    border: grey solid 1px;
    border-radius: 15px;
    overflow: hidden;
  }
</style>