<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="规则名称">
              <a-input placeholder="please enter 规则名称" v-model="queryParam.ruleName"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="规则Code">
              <a-input placeholder="please enter 规则Code" v-model="queryParam.ruleCode"/>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">New</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('编码校验规则')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            Delete
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> Bulk Operations
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <a-alert type="info" showIcon style="margin-bottom: 16px;">
      <template slot="message">
        <span>selected</span>
        <a style="font-weight: 600;padding: 0 4px;">{{ selectedRowKeys.length }}</a>
        <span>item</span>
        <template v-if="selectedRowKeys.length>0">
          <a-divider type="vertical"/>
          <a @click="onClearSelected">clear</a>
        </template>
      </template>
    </a-alert>

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
      @change="handleTableChange">

      <template slot="action" slot-scope="text, record">
        <a @click="handleEdit(record)">Edit</a>
        <a-divider type="vertical"/>
        <a @click="handleTest(record)">功能测试</a>
        <a-divider type="vertical"/>
        <a-dropdown>
          <a class="ant-dropdown-link">
            <span>更多</span>
            <a-icon type="down"/>
          </a>
          <a-menu slot="overlay">
            <a-menu-item>
              <a-popconfirm title="Are you sure you want to delete it?" @confirm="handleDelete(record.id)">Delete</a-popconfirm>
            </a-menu-item>
          </a-menu>
        </a-dropdown>
      </template>

    </a-table>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <sys-check-rule-modal ref="modalForm" @ok="modalFormOk"/>

    <sys-check-rule-test-modal ref="testModal"/>

  </a-card>
</template>

<script>
  import JEllipsis from '@/components/jeecg/JEllipsis'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import SysCheckRuleModal from './modules/SysCheckRuleModal'
  import SysCheckRuleTestModal from './modules/SysCheckRuleTestModal'

  export default {
    name: 'SysCheckRuleList',
    mixins: [JeecgListMixin],
    components: { SysCheckRuleModal, SysCheckRuleTestModal, JEllipsis },
    data() {
      return {
        description: '编码校验规则管理page面',
        // 表头
        columns: [
          {
            title: '#',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: (t, r, i) => i + 1
          },
          {
            title: '规则名称',
            align: 'center',
            dataIndex: 'ruleName'
          },
          {
            title: '规则Code',
            align: 'center',
            dataIndex: 'ruleCode'
          },
          {
            title: '规则描述',
            align: 'center',
            dataIndex: 'ruleDescription',
            customRender: (t) => (<j-ellipsis value={t} length={48}/>)
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: 'center',
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: '/sys/checkRule/list',
          delete: '/sys/checkRule/delete',
          deleteBatch: '/sys/checkRule/deleteBatch',
          exportXlsUrl: 'sys/checkRule/exportXls',
          importExcelUrl: 'sys/checkRule/importExcel',
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      handleTest(record) {
        this.$refs.testModal.open(record.ruleCode)
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>