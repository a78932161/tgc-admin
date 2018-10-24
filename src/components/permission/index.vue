<template>
  <div class="app-container">


    <form-container :model="param" class="_search">
      <field-input class="searchInput" label="搜索关键字" :wrapforlike="true" v-model="param.namelike"></field-input>
      <el-button type="primary" style="margin-left: 15px">查询</el-button>
      <el-button type="primary" @click="dialogShow('add')">增加角色</el-button>
    </form-container>


    <el-table :data="list" class="_list"
              v-loading.body="$apollo.queries.list.loading" element-loading-text="Loading"
              fit highlight-current-row
    >
      <bos-table-column lable="编号" field="number" width="180"></bos-table-column>
      <bos-table-column lable="角色名" field="name"></bos-table-column>
      <bos-table-column lable="创建时间" field="createtime" width="200"
                        :filter="(row)=>formatTime(row.createtime)"
      ></bos-table-column>
      <bos-table-column lable="状态" field="disabled"
                        :filter="(row)=>adminStatus(row.disabled)"
      ></bos-table-column>
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="dialogShow('edit',scope.row)">修改</el-button>
          <el-button type="text" size="mini" @click="deleteOne(scope.row)" class="_remote-tip">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <bos-paginator :pageInfo="paginator" @bosCurrentPageChanged="changePage"/>
    <field-dialog :title="title" ref="dialog" @confirm="dialogConfirm">
      <form-container ref="inlineForm" :model="inlineForm">
        <field-input v-model="inlineForm.name" label="角色名" width="10"
                     :rules="r(true).all(R.require)" prop="name"></field-input>
        <el-form-item label="权限" label-width="120px">
          <checkbox1 all="全选" :list="list1" :chosen="chosen" @dataList="dataList"></checkbox1>
        </el-form-item>
      </form-container>
    </field-dialog>


  </div>
</template>

<script>


  import {filterAttr} from 'common/js'
  import {rules} from 'common/js/validate';
  import api from 'graph/role.graphql';
  import {historyPageMixin} from 'common/js/mixin';
  import checkbox1 from '@/base/checkbox'

  export default {
    components: {
      checkbox1
    },
    data() {
      return {
        rules,
        dialogType: '',
        inlineForm: {},
        list1: ['用户管理', '角色管理', '订单管理', '商品管理'],
        chosen: [],
      }
    },
    mixins: [historyPageMixin],
    apollo: {
      list() {//loadingKey
        //created的时候会执行一次，context代表的是vm对象，调试时可以查阅代码：vue-apollo.esm.js:  options = options.call(context)
        return this.getEntityQuery(api.RoleList);
      }
    },
    computed: {
      isDialogAdd() {
        return this.dialogType === 'add' ? true : false;
      },
      title() {
        return this.dialogType === 'add' ? '添加角色' : '修改角色';
      }
    },

    methods: {
      dataList(data) {
        console.log(data);
      },
      formatTime(time) {
        if (time == null) return;
        return new Date(parseInt(time)).toLocaleString().replace(/:\d{1,2}$/, ' ');
      },
      adminStatus(disabled) {
        return disabled ? '删除' : "正常";
      },
      dialogShow(type, row) {
        this.dialogType = type;
        this.$refs.dialog.show();
        this.$nextTick(() => {    //dialog框出现以后，进行清空验证
          this.$refs.inlineForm.clearValidate();
        });
        if (type === 'add') {
          this.inlineForm = {};
          return;
        }
        this.inlineForm = JSON.parse(JSON.stringify(row));
        this.inlineForm.password = '';
      },

      dialogConfirm() {
        this.$refs.inlineForm.gqlValidate(this.dialogType === 'add' ? api.addAdminist : api.editAdministPassWord, {
          administ: filterAttr(this.inlineForm, 'repeatPassword')
        }, () => {
          this.callback(`${this.title}成功`);
          this.$refs.dialog.hide();
        });
      },
      deleteOne(row) {
        let item = JSON.parse(JSON.stringify(row));
        item.disabled = true;
        this.gqlMutate(api.editAdminist, {
          administ: filterAttr(item)
        }, () => {
          this.callback('删除成功')
        })
      }
    }
  }
</script>

