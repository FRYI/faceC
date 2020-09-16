<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->

    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('torder')">ExportXls</a-button>
      <a-upload name="file"  :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">Import</a-button>
      </a-upload>
      <j-super-query :fieldList="fieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery">

      </j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>Delete</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">Clear Selected</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            download
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">Edit</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">More <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="confirm Delete?" @confirm="() => handleDelete(record.id)">
                  <a>Delete</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <torder-modal ref="modalForm" @ok="modalFormOk"></torder-modal>
    <supplier-modal ref="smodalForm" @ok="modalFormOk"></supplier-modal>
    <client-modal ref="cmodalForm" @ok="modalFormOk"></client-modal>
    <product-modal ref="modalFormp" :immm="imgdata" @ok="modalFormOk"></product-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import TorderModal from './modules/TorderModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import SupplierModal from './modules/SupplierModal'
  import ClientModal from './modules/ClientModal'
  import ProductModal from './modules/ProductModal'
  import { getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JSuperQuery from '@/components/jeecg/JSuperQuery'
  import {filterObj} from "../../utils/util";
  import ProjectModal from "./modules/ProjectModal";




  const superQueryFieldList=[
    {
      type: "string",
      value: "Torder.ordernumber",
      text: "Order Number"
    },
    {
      type: "string",
      value: "Torder.supplier",
      text: "Supplier"
    },
    {
      type: "string",
      value: "Torder.client",
      text: "Client"
    },
    {
      type: "string",
      value: "Torder.chop",
      text: "Chop"
    },
    {
      value:'Torder.season',
      type:"string",
      text: 'season'
    },
    {
      type: "string",
      value: "Torder.product",
      text: "Product"
    },
    {
      type: "string",
      value: "Torder.incoterm",
      text: "Incoterm"
    },
    {
      type: "string",
      value: "Torder.delivery",
      text: "Delivery"
    },
    {
      type: "int",
      value: "Torder.priceUsd",
      text: "Price Usd"
    },
    {
      type: "string",
      value: "Torder.client",
      text: "Client"
    },
    {
      value:'Torder.quantity',
      type: "int",
      text: 'Quantity'
    },
    {
      value:'Torder.amountRmb',
      type: "int",
      text: 'Amount Rmb'
    },
    {
      value:'Torder.amountUsd',
      type: "int",
      text: 'Amount Usd'
    },
    {
      value:'Torder.depositRmb',
      type: "int",
      text: 'Deposit Rmb'
    },
    {
      value:'Torder.depositUsd',
      type: "int",
      text: 'Deposit Usd'
    },
    {
      value:'Torder.depositPay',
      type: "int",
      text: 'Deposit Pay'
    },
    {
      value:'Torder.balanceRmb',
      type: "int",
      text: 'Balance Rmb'
    },
    {
      value:'Torder.balanceUsd',
      type: "int",
      text: 'Balance Usd'
    },
    {
      value:'Torder.balancePay',
      type: "int",
      text: 'Balance Pay'
    },
    {
      value:'Torder.deliveryDate',
      type: "date",
      text: 'Delivery Date',
    },
    {
      value:'Torder.deliverySituation',
      type: "string",
      text: 'Delivery Situation'
    },
    {
      value:'Torder.etd',
      type: "date",
      text: 'Etd',

    },
    {
      value:'Torder.delay',
      type: "int",
      text: 'Delay'
    },
    {
      value:'Torder.eta',
      type: "date",
      text: 'Eta',

    },
    {
      value:'Torder.status',
      type: "stirng",
      text: 'Status'
    },
    {
      value:'Torder.statusdate',
      type: "string",
      text: 'Status Date',

    },
    {
      value:'Torder.createTime',
      type: "date",
      text: 'Create Time',

    },
    {
      value:'Torder.updateTime',
      type: "date",
      text: 'Update Time',

    },


    {
      type: "string",
      value: "Product.sku",
      text: "Product.Sku"
    },
    {
      type: "string",
      value: "Product.project",
      text: "Product.Project"
    },
    {
      type: "string",
      value: "Product.supplier",
      text: "Product.Supplier"
    },
    {
      type: "string",
      value: "Product.productName",
      text: "Product.Product Name"
    },

    {
      type: "string",
      value: "Product.paramData",
      text: "Product.Specification"
    },
    {
      type: "string",
      value: "Product.description",
      text: "Product.Description"
    },
    {
      type: "date",
      value: "Product.createTime",
      text: "Product.Create Time"
    },
    {
      type: "date",
      value: "Product.updateTime",
      text: "Product.Update Time"
    }


  ]

  export default {
    name: "TorderList",
    mixins:[JeecgListMixin, mixinDevice],

    components: {
      JDate,
      TorderModal,
      SupplierModal,
      ClientModal,
      ProductModal,
      JSuperQuery,
    },
    data () {
      return {
        imgdata:"",
        description: 'torder管理页面',
        fieldList: superQueryFieldList,
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'Ordernumber',
            align:"center",
            dataIndex: 'ordernumber'
          },
          {
            title:'Supplier',
            align:"center",
            dataIndex: 'supplier',
            customRender: (text, row, index) => {
              const show =()=>{
                this.showInfo(row,"supplier");  //调用method里面的方法
              }
              let content = (<a href="javascript:;"  onClick={ show }>{text}</a>);
              const obj = {
                children: content,
                attrs: { colSpan: 1 },
              };
              return obj;
            }
          },

          {
            title:'Client',
            align:"center",
            dataIndex: 'client',
            customRender: (text, row, index) => {
              const show =()=>{
                this.showInfo(row,"client");  //调用method里面的方法
              }
              let content = (<a href="javascript:;"  onClick={ show }>{text}</a>);
              const obj = {
                children: content,
                attrs: { colSpan: 1 },
              };
              return obj;
            }
          },
          {
            title:'Chop',
            align:"center",
            dataIndex: 'chop'
          },
          {
            title:'Season',
            align:"center",
            dataIndex: 'season'
          },
          {
            title:'Product',
            align:"center",
            dataIndex: 'product',
            customRender: (text, row, index) => {


              const show =()=>{
                this.showInfo(row,"product");  //调用method里面的方法
              }
              let content = (<a href="javascript:;"  onClick={ show }>{text}</a>);
              const obj = {
                children: content,
                attrs: { colSpan: 1 },
              };
              return obj;
            }

           },

          {
            title:'Incoterm',
            align:"center",
            dataIndex: 'incoterm'
          },
          {
            title:'Delivery',
            align:"center",
            dataIndex: 'delivery'
          },
          {
            title:'Price Usd',
            align:"center",
            dataIndex: 'priceUsd'
          },
          {
            title:'Quantity',
            align:"center",
            dataIndex: 'quantity'
          },
          {
            title:'Amount Rmb',
            align:"center",
            dataIndex: 'amountRmb'
          },
          {
            title:'Amount Usd',
            align:"center",
            dataIndex: 'amountUsd'
          },
          {
            title:'Deposit Rmb',
            align:"center",
            dataIndex: 'depositRmb'
          },
          {
            title:'Deposit Usd',
            align:"center",
            dataIndex: 'depositUsd'
          },
          {
            title:'Deposit Pay',
            align:"center",
            dataIndex: 'depositPay'
          },
          {
            title:'Balance Rmb',
            align:"center",
            dataIndex: 'balanceRmb'
          },
          {
            title:'Balance Usd',
            align:"center",
            dataIndex: 'balanceUsd'
          },
          {
            title:'Balance Pay',
            align:"center",
            dataIndex: 'balancePay'
          },
          {
            title:'Delivery Date',
            align:"center",
            dataIndex: 'deliveryDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Delivery Situation',
            align:"center",
            dataIndex: 'deliverySituation'
          },
          {
            title:'Etd',
            align:"center",
            dataIndex: 'etd',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Delay',
            align:"center",
            dataIndex: 'delay'
          },
          {
            title:'Eta',
            align:"center",
            dataIndex: 'eta',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Status',
            align:"center",
            dataIndex: 'status'
          },
          {
            title:'Statusdate',
            align:"center",
            dataIndex: 'statusdate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Create Time',
            align:"center",
            dataIndex: 'createTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Update Time',
            align:"center",
            dataIndex: 'updateTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title: 'Action',
            dataIndex: 'action',
            align:"center",
            // fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/torder/torder/list",
          delete: "/torder/torder/delete",
          deleteBatch: "/torder/torder/deleteBatch",
          exportXlsUrl: "/torder/torder/exportXls",
          importExcelUrl: "torder/torder/importExcel",
        },
        dictOptions:{},
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      getQueryParams(){
        //高级查询器
        let sqp = {}
        if(this.superQueryParams){
          sqp['superQueryParams']=encodeURI(this.superQueryParams)
          sqp['superQueryMatchType'] = this.superQueryMatchType
        }
        var param = Object.assign(sqp, this.queryParam, this.isorter ,this.filters);

        param.field = this.getQueryField();
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        delete param.birthdayRange; //范围参数不传递后台
        return filterObj(param);
      },
      initDictConfig(){

      },

      showInfo(text,name){
        console.log(text)
        let value ={}

        name=="supplier"?value = {

          supplier:text.supplier
        }:(name=="client"?value = {
          client:text.client
        }:value = {
          product:{"season":text.season,"product":text.product}
        })

        var link = "/"+name+"/"+name+"/queryByName"
        getAction(link,value).then((res)=>{
          if(res.success){
            console.log(res)
            this.edit11(res.result,name)
          }else{
            console.log(res.message);
          }
        })
      },
      edit11 (obj,name) {
        var ll,ls
        if(name == "supplier"){
          ll =  this.$refs.smodalForm
          ls = { 'pin':obj.pin, 'supplier':obj.supplier, 'contact':obj.contact, 'tel':obj.tel, 'email':obj.email, 'website':obj.website, 'city':obj.city, 'address':obj.address, 'zipCode':obj.zipCode, 'scope':obj.scope}
        }else if (name=="client"){
          ll=this.$refs.cmodalForm
          ll.model = Object.assign({}, obj);
          ls ={'client':obj.client,'contact':obj.contact,'tel':obj.tel,'email':obj.email,'website':obj.website,'city':obj.city,'address':obj.address,'zipCode':obj.zipCode}
        }else if (name =="product") {
          ll= this.$refs.modalFormp
          ll.model = Object.assign({}, obj);
          ls ={'season':obj.season,'sku':obj.sku,'project':obj.project,'productName':obj.productName,'supplier':obj.supplier,'paramData':obj.paramData,'description':obj.description,'photoString':obj.photoString}
          console.log(ll)
          ll.array =JSON.parse(obj.paramData)
          console.log(ll.array)
          ll.imgsrc = obj.photoString
        }

        ll.form.resetFields();
        ll.model.id = obj.id
        ll.visible = true;
        ll.$nextTick(() => {

        //   this.$refs.smodalForm.form.setFieldsValue({ 'pin':obj.pin, 'supplier':obj.supplier, 'contact':obj.contact, 'tel':obj.tel, 'email':obj.email, 'website':obj.website, 'city':obj.city, 'address':obj.address, 'zipCode':obj.zipCode, 'scope':obj.scope})
          ll.form.setFieldsValue(ls)

          // this.$refs.smodalForm.form.setFieldsValue({'supplier': "it"})

        })


      }

      }







  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>