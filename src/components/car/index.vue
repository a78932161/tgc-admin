<template>
  <div>
    <div class="app-container">
      <form-container :model="param" class="_search">
        <field-input class="searchInput" label="搜索关键字" :wrapforlike="true" v-model="param.namelike"></field-input>
        <el-button type="primary" style="margin-left: 15px">查询</el-button>
        <el-button type="primary" @click="domeClick('add')">假装进入</el-button>
        <addCar @on-selected-file='addCars' style="margin-left: 10px"></addCar>
      </form-container>
    </div>
    <el-table :data="list" class="_list" v-loading.body="$apollo.queries.list.loading" element-loading-text="Loading"
              fit highlight-current-row>
      <bos-table-column lable="编号" field="number"/>
      <bos-table-column lable="品牌" field="brand"/>
      <bos-table-column lable="型号" field="model"/>
      <bos-table-column lable="厂家指导价" field="price"/>
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">

        </template>
      </el-table-column>
    </el-table>
    <bos-paginator :pageInfo="paginator" @bosCurrentPageChanged="changePage"/>
  </div>
</template>

<script>
  import {historyPageMixin} from 'common/js/mixin';
  import api from 'graph/car.graphql'
  import addCar from 'base/Excel/index'
  import {baseURL} from "@/api/config";
  import request from 'common/js/request';

  export default {
    name: "index",
    mixins: [historyPageMixin],
    components: {
      addCar
    },
    apollo: {
      list() {
        return this.getEntityQuery(api.getCarInfoList);
      }
    },
    data() {
      return {
        param: {},
      }
    },
    methods: {
      addCars(data) {
        let loadingInstance = this.$loading({fullscreen: true});
        data.results.forEach((value, index) => {
          let arr = [];
          for (let key in value) {
            arr.push({key: key, value: value[key]}); //属性
          }
          let b = JSON.stringify(arr);
          let c = JSON.parse(b);
          request({
            url: `${baseURL}/sts`
          }).then((result) => {
            let credentials = result.assumeRoleResponse.credentials;
            const client = new OSS({
              endpoint: result.endpoint,
              bucket: result.bucketName,
              accessKeyId: credentials.accessKeyId,
              accessKeySecret: credentials.accessKeySecret,
              stsToken: credentials.securityToken,
            });
            const storeAs = result.resourceId;
            client.put(storeAs, new Buffer(b)).then((results) => {
              this.form = {
                model: c[0].value,
                brand: c[3].value,
                price: c[1].value,
                filename: results.name,
              };
              this.$apollo.mutate({
                mutation: api.carInfo_add,
                variables: {carInfo: this.form}
              }).then((res) => {
                if (data.results.length == index + 1) {
                  loadingInstance.close();
                  this.$forceUpdate();
                  this.$message.success('导入成功');
                }
              }).catch((res) => {
                this.$message.error(`${res}`);
              })
            })
          });
        });
      },
      domeClick(type, row) {
        if (type === 'add') {
          this.$router.push({path: 'carInfo', query: {type: type}});
        } else {
          this.$router.push({path: 'carInfo', query: {id: row.id, type: type}});
        }
      }
    }
  }
</script>

<style scoped>

</style>
