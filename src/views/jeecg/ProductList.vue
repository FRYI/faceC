<template>
  <a-card :bordered="false">

    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('product')">ExportXls</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
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
            title:'season',
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

              var json2 = JSON.parse(text)
              return (
                <div>
                  {Object.keys(json2).map((obj, idx) => (
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