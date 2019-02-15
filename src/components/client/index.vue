<template>
  <div>
    <div class="app-container">
      <form-container :model="param" class="_search">
        <field-input class="searchInput" label="搜索关键字" :wrapforlike="true" v-model="param.namelike"></field-input>
        <el-button type="primary" style="margin-left: 15px">查询</el-button>
        <!--<el-button type="primary">增加车辆</el-button>-->
      </form-container>
    </div>
    <el-table :data="list" class="_list" v-loading.body="$apollo.queries.list.loading" element-loading-text="Loading"
              fit highlight-current-row>
      <bos-table-column lable="编号" field="number"/>
      <bos-table-column lable="客户" field="name"/>
      <bos-table-column lable="手机号" field="phone"/>
      <bos-table-column lable="积分" field="integral"/>
      <bos-table-column lable="门店" field="store.name"/>
      <bos-table-column lable="邀请人" field="invitedId"></bos-table-column>
      <bos-table-column lable="销售顾问" field="salesConsultantAdmin.name"></bos-table-column>
    </el-table>
    <bos-paginator :pageInfo="paginator" @bosCurrentPageChanged="changePage"/>
  </div>
</template>

<script>
  import {historyPageMixin} from 'common/js/mixin';
  import api from 'graph/client.graphql'

  export default {
    name: "index",
    mixins: [historyPageMixin],
    apollo: {
      list() {
        return this.getEntityQuery(api.getCustomerList);
      }
    },
    data() {
      return {
        param: {},
      }
    }
  }
</script>

<style scoped>

</style>
