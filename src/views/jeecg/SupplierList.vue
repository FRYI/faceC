<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->

    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('supplier')">ExportXls</a-button>
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

    <supplier-modal ref="modalForm" @ok="modalFormOk"></supplier-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import SupplierModal from './modules/SupplierModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import JSuperQuery from '@/components/jeecg/JSuperQuery'
  import {filterObj} from "../../utils/util";
  import ProjectModal from "./modules/ProjectModal";



  const superQueryFieldList=[
    {
      type: "string",
      value: "pin",
      text: "Pin"
    },
    {
      type: "string",
      value: "supplier",
      text: "Supplier"
    },
    {
      type: "string",
      value: "contact",
      text: "Contact"
    },
    {
      type: "string",
      value: "tel",
      text: "Tel"
    },
    {
      type: "string",
      value: "email",
      text: "Email"
    },
    {
      type: "string",
      value: "website",
      text: "Website"
    },
    {
      type: "string",
      value: "city",
      text: "City"
    },
    {
      type: "string",
      value: "address",
      text: "Address"
    },
    {
      type: "string",
      value: "zipCode",
      text: "Zip Code"
    },
    {
      type: "string",
      value: "scope",
      text: "Scope"
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
    name: "SupplierList",
    mixins:[JeecgListMixin, mixinDevice],

    components: {
      JDate,
      SupplierModal,
      JSuperQuery,
    },
    data () {
      return {
        description: 'supplier管理页面',
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
            title:'Pin',
            align:"center",
            dataIndex: 'pin'
          },
          {
            title:'Supplier',
            align:"center",
            dataIndex: 'supplier'
          },
          {
            title:'Contact',
            align:"center",
            dataIndex: 'contact'
          },
          {
            title:'Tel',
            align:"center",
            dataIndex: 'tel'
          },
          {
            title:'Email',
            align:"center",
            dataIndex: 'email'
          },
          {
            title:'Website',
            align:"center",
            dataIndex: 'website'
          },
          {
            title:'City',
            align:"center",
            dataIndex: 'city'
          },
          {
            title:'Address',
            align:"center",
            dataIndex: 'address'
          },
          {
            title:'Zip Code',
            align:"center",
            dataIndex: 'zipCode'
          },
          {
            title:'Scope',
            align:"center",
            dataIndex: 'scope'
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
          list: "/supplier/supplier/list",
          delete: "/supplier/supplier/delete",
          deleteBatch: "/supplier/supplier/deleteBatch",
          exportXlsUrl: "/supplier/supplier/exportXls",
          importExcelUrl: "supplier/supplier/importExcel",
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
</style>