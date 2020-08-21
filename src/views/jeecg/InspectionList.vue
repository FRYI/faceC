<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->

    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">Add</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('inspection')">ExportXls</a-button>
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
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>Delete</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <inspection-modal ref="modalForm" @ok="modalFormOk"></inspection-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import InspectionModal from './modules/InspectionModal'
  import JSuperQuery from '@/components/jeecg/JSuperQuery'
  import {filterObj} from "../../utils/util";


  const superQueryFieldList=[
    {
      type: "string",
      value: "ordernumber",
      text: "Order Number"
    },
    {
      type: "string",
      value: "season",
      text: "season"
    },
    {
      type: "string",
      value: "supplier",
      text: "Supplier"
    },
    {
      type: "string",
      value: "inspector",
      text: "Inspector"
    },
    {
      type: "date",
      value: "depositeDate",
      text: "Deposite Date"
    },
    {
      type: "date",
      value: "comfirmDate",
      text: "Comfirm Date"
    },
    {
      type: "date",
      value: "inspectionDate",
      text: "Inspection Date"
    },
    {
      type: "date",
      value: "deliveryDate",
      text: "Delivery Date"
    },
    {
      type: "int",
      value: "amount",
      text: "Amount"
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
    name: "InspectionList",
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      InspectionModal,
      JSuperQuery,
    },
    data () {
      return {
        description: 'inspection',
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
            title:'Order Number',
            align:"center",
            dataIndex: 'ordernumber'
          },
          {
            title:'Supplier',
            align:"center",
            dataIndex: 'supplier'
          },
          {
            title:'Inspector',
            align:"center",
            dataIndex: 'inspector'
          },
          {
            title:'Deposite Date',
            align:"center",
            dataIndex: 'depositeDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Comfirm Date',
            align:"center",
            dataIndex: 'comfirmDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'Inspection Date',
            align:"center",
            dataIndex: 'inspectionDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
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
            title:'Amount',
            align:"center",
            dataIndex: 'amount'
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
          list: "/inspection/inspection/list",
          delete: "/inspection/inspection/delete",
          deleteBatch: "/inspection/inspection/deleteBatch",
          exportXlsUrl: "/inspection/inspection/exportXls",
          importExcelUrl: "inspection/inspection/importExcel",
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