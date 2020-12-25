<template>
  <a-card :bordered="false">

    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('product')">ExportXls</a-button>
      <a-upload name="file" :data="fileData" :beforeUpload="catchData" ref="aupload" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">Edit</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">More <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="confirm删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>Delete</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <product-modal ref="modalForm" @ok="modalFormOk"></product-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ProductModal from './modules/ProductModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import JSuperQuery from '@/components/jeecg/JSuperQuery'
  import {filterObj} from "../../utils/util";
  import {stream} from "exceljs";
  import {buffer} from "ant-design-vue";
  const Excel = require('exceljs')
  const XLSX = require("xlsx")



  const superQueryFieldList=[
    {
      text:'season',
      type:"string",
      value: 'season'
    },
    {
      type: "string",
      value: "sku",
      text: "Sku"
    },
    {
      type: "string",
      value: "project",
      text: "Project"
    },
    {
      type: "string",
      value: "supplier",
      text: "Supplier"
    },
    {
      type: "string",
      value: "productName",
      text: "Product Name"
    },
    {
      type: "string",
      value: "paramData",
      text: " Specification"
    },
    {
      type: "string",
      value: "description",
      text: "Description"
    },
    {
      type: "date",
      value: "createTime",
      text: "Create Time"
    },
    {
      type: "date",
      value: "updateTime",
      text: "Update Time"
    }
  ]
  export default {
    name: "ProductList",
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      JDate,
      ProductModal,
      JSuperQuery,
    },
    data () {
      return {
        description: 'product',
        fileData: {},
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
            title:'Season',
            align:"center",
            dataIndex: 'season'
          },
          {
            title:'Sku',
            align:"center",
            dataIndex: 'sku'
          },
          {
            title:'Project',
            align:"center",
            dataIndex: 'project'
          },
          {
            title:'Product Name',
            align:"center",
            dataIndex: 'productName'
          },
          {
            title:'Supplier',
            align:"center",
            dataIndex: 'supplier'
          },
          {
            title:'Specification',
            align:"center",
            dataIndex: 'paramData',
            customRender:function (text) {
              console.log(text)
              var json2 = JSON.parse(text)
             if(json2 == null){
               return text
             }
              return (
                <div>
                  {
                    Object.keys(json2).map((obj, idx) => (
                    <div> <span style="margin-right:5px;">{obj}</span> <span style="font-weight:bold;font-style: oblique;">{json2[obj]}</span></div>
                  ))}
                </div>
            )

             }

          },
          {
            title:'Description',
            align:"center",
            dataIndex: 'description'
          },
          {
            title:'Photo',
            align:"center",
            dataIndex: 'photoString',
            customRender:function (text) {
              return  <img src= {text}  width="auto" height="80px" />
            }
          },
          {
            title:'Created',
            align:"center",
            dataIndex: 'createTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Updated',
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
          list: "/product/product/list",
          delete: "/product/product/delete",
          deleteBatch: "/product/product/deleteBatch",
          exportXlsUrl: "/product/product/exportXls",
          importExcelUrl: "product/product/importExcel",
        },
        dictOptions:{},
      }
    },
    watch:{
      // fileData(val){
      //   this.postData(val)
      //   alert("111")
      //
      // }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      async catchData(file, fileList) {

      let  blobUpdate = await this.fileChange(file)
        return Promise.resolve(new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}))
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
                worksheet1.getCell('F4').value=""
                worksheet1.getRow(item.range.br.nativeRow+1).getCell(item.range.br.nativeCol+1).value = base64
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
                  let base64 = worksheet1.getRow(i).getCell(8).value
                console.log(base64)
                  if(base64 && base64.startsWith("data:image")) {
                    worksheet1.getRow(i).getCell(8).value = ""
                    const imageId = workbook.addImage({
                      base64: base64,
                      extension: 'jpg',
                    });
                    worksheet1.addImage(imageId, {
                      tl: {col: 8-1, row: i-1},
                      ext: {width: 50, height: 50}
                    });
                  }
              }
              workbook.xlsx.writeBuffer().then(data => {
                blobUpdate = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
                console.log(blobUpdate)
                // this.fileData = new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'})

                resolve(blobUpdate)
                // return Promise.resolve(new window.File([blobUpdate], "product.xlsx", {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}))
              });
            }).catch((error)=> {
              console.log("readFile fail", error);
            })
          };
        })
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
      }


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
  .imgsty{
    width: 20px;
    height: auto;
  }
  td{
    height: 100px;
  }
  img{
    margin-top: 0;
  }
</style>