<template>
  <div>
    <div class="app-container">
      <form-container :model="param" class="_search">
        <field-input class="searchInput" label="搜索关键字" :wrapforlike="true" v-model="param.namelike"></field-input>
        <el-button type="primary" style="margin-left: 15px">查询</el-button>
        <el-button type="primary" @click="dialogShow('add')">增加门店</el-button>
      </form-container>
    </div>
    <el-table :data="list" class="_list" v-loading.body="$apollo.queries.list.loading" element-loading-text="Loading"
              fit highlight-current-row>
      <bos-table-column lable="编号" field="number"/>
      <bos-table-column lable="店名" field="name"/>
      <bos-table-column lable="联系方式" field="phone"/>
      <bos-table-column lable="地址" field="address"/>
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="dialogShow('edit',scope.row)">编辑</el-button>
          <el-button type="text" size="mini" v-if="scope.row.disabled==false" @click="openClose(scope.row)">禁用
          </el-button>
          <el-button type="text" size="mini" v-if="scope.row.disabled==true" @click="openClose(scope.row)">开启
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <bos-paginator :pageInfo="paginator" @bosCurrentPageChanged="changePage"/>

  </div>
</template>

<script>
  import {historyPageMixin} from 'common/js/mixin';
  import api from 'graph/shop.graphql'


  export default {
    name: "index",
    mixins: [historyPageMixin],
    apollo: {
      list() {
        return this.getEntityQuery(api.getStoreList);
      }
    },
    data() {
      return {
        param: {},
        title: "",
        inlineForm: {},
      }
    },
    methods: {
      dialogShow(type, row) {
        if (type === 'add') {
          this.$router.push({path: 'shop', query: {type: type}});
        } else {
          this.$router.push({path: 'shop', query: {id: row.id, type: type}});
        }
      },
      openClose(row) {
        row.disabled = !row.disabled;
        console.log(row);
        this.$apollo.mutate({
          mutation: api.store_modify,
          variables: {store: row},
        }).then(() => {
          this.$message.success('修改成功');
        })
      }
    },
  }
</script>

<style scoped>

</style>
