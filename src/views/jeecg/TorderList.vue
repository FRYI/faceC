<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->

    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('torder')"  :loading="loading">ExportXls</a-button>
      <a-upload name="file" :beforeUpload="catchData" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">Import</a-button>
      </a-upload>
      <j-super-query :fieldList="fieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery">

      </j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>Delete</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> Bulk Operations <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> Selected<a style="font-weight: 600">{{ selectedRowKeys.length }}</a>item
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
  const Excel = require('exceljs')
  const XLSX = require("xlsx")



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
        loading: false,
        imgdata:"",
        description: 'torder管理page面',
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
      changeloading(data){
        alert("////")
        this.loading= true;
      },
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
      downloadBlob(filename,blob1){
        // 创建隐藏的可下载链接
        var eleLink = document.createElement('a');
        eleLink.download = filename;
        eleLink.style.display = 'none';
        // 字符内容转变成blob地址
        // var blob1= new Blob([content1]);
        eleLink.href = URL.createObjectURL(blob1);
        // 触发点击
        document.body.appendChild(eleLink);
        eleLink.click();
        // 然后移除
        document.body.removeChild(eleLink);
      },
      async catchData(file, fileList) {
        let  blobUpdate = await this.fileChange(file)
               this.downloadBlob("torderBeforeUpdateCheck.xlsx",blobUpdate)
         return Promise.resolve(new window.File([blobUpdate], "torder.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}))
      },
      uint8arrayToBase64(u8Arr) {
        let CHUNK_SIZE = 0x8000; //arbitrary number
        let index = 0;
        let length = u8Arr.length;
        let result = '';
        let slice;
        while (index < length) {
          slice = u8Arr.subarray(index, Math.min(index + CHUNK_SIZE, length));
          result += String.fromCharCode.apply(null, slice);
          index += CHUNK_SIZE;
        }
        return "data:image/png;base64," + btoa(result);
      },
      fileChange(file){
        let vm = this
        let blobUpdate = null
        const workbook = new Excel.Workbook();
        return new Promise((resolve, reject) => {
          var fileReader = new FileReader();
          fileReader.readAsArrayBuffer(file);
          fileReader.onload = (e) => {
            const buffer = e.target.result;
            workbook.xlsx.load(buffer).then(async (wb)=> {
              console.log("readFile success");
              let worksheet1 = wb.getWorksheet(1)


              worksheet1.getImages().forEach((item) =>{
                let img = wb.getImage(Number(item.imageId))
                let base64 = vm.uint8arrayToBase64(img.buffer)
                console.log(base64)
                worksheet1.getCell('A4').value=""
                console.log(item);
                worksheet1.getRow(item.range.tl.nativeRow+1).getCell(item.range.tl.nativeCol+1).value = base64
              })
              workbook.xlsx.writeBuffer().then(data => {
                blobUpdate = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
                console.log(blobUpdate)
                this.fileData = new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'})

                resolve(blobUpdate)
                // return Promise.resolve(new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}))
              });
            }).catch((error)=> {
              console.log("readFile fail", error);
            })
          };
        })
      },
      fileChange2(file){
        alert("downloading")
        let vm = this
        let blobUpdate = null
        const workbook = new Excel.Workbook();
        return new Promise((resolve, reject) => {
          var fileReader = new FileReader();
          fileReader.readAsArrayBuffer(file);
          fileReader.onload = (e) => {
            const buffer = e.target.result;
            workbook.xlsx.load(buffer).then(async (wb)=> {
              console.log("readFile success");
              let worksheet1 = wb.getWorksheet(1)

              var rowCount = worksheet1.rowCount
              console.log(rowCount)
              for(var i =1;i<= rowCount;i++){
                console.log(i)
                let base64 = worksheet1.getRow(i).getCell(1).value
                // console.log(base64)
                if(base64 && base64.startsWith("data:image")) {
                  worksheet1.getRow(i).getCell(1).value = ""
                  const imageId = workbook.addImage({
                    base64: base64,
                    extension: 'jpg',
                  });
                  worksheet1.addImage(imageId, {
                    tl: {col: 0, row: i-1},
                    ext: {width: 50, height: 50}
                  });
                }
              }
              workbook.xlsx.writeBuffer().then(data => {
                blobUpdate = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
                // console.log(blobUpdate)
                // this.fileData = new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'})

                resolve(blobUpdate)
                // return Promise.resolve(new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}))
              });
            }).catch((error)=> {
             alert("write fail");
            })
          };
        })
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
          ll = this.$refs.modalFormp
          ll.model = Object.assign({}, obj);
          ls = {
            'season': obj.season,
            'sku': obj.sku,
            'project': obj.project,
            'productName': obj.productName,
            'supplier': obj.supplier,
            'paramData': obj.paramData,
            'description': obj.description,
            'photoString': obj.photoString
          }
          console.log(ll)
          ll.imgsrc = obj.photoString
          console.log(obj.paramData)
          if (obj.paramData != null && !obj.paramData.match(".*null.*")) {

            ll.array = JSON.parse(obj.paramData)
            console.log(ll.array)
            var json2 = JSON.parse(obj.paramData)
            Object.keys(json2).map((obj, idx) => (
              setTimeout(() => {
                var cs = "paramData[" + obj + "]".toString()
                // this.form.setFieldsValue({ ["paramData[color]"]:json2[obj]});
                ll.form.setFieldsValue({[cs]: json2[obj]});
              }, 1000)
            ))
          }else {
            ll.form.setFieldsValue({["paramData[color]"]:""});
          }
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