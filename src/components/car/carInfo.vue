<template>
  <div class="bg">
    <el-card class="card-box">
      <form-container ref="form" :model="form" class="form-container" :_readonly="readonly">
        <field-input label="品牌" v-model="form.name" width="2.5" prop="name"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="型号" v-model="form.name" width="2.5" prop="name"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="厂商指导价" v-model="form.name" width="2.5" prop="name"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="权重" v-model="form.name" width="2.5" prop="name"
                     :rules="r(true).all(R.require)"></field-input>
        <field-image label="图片" v-model="form.name" width="2.5" prop="name"></field-image>
      </form-container>


      <div class="button-container">
        <el-button type="primary" @click="backClick">返回</el-button>
        <el-button type="primary" @click="submit" v-show="!readonly">确 定</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
  export default {
    name: "carinfo",
    data() {
      return {
        param: {id: ''},
        form: {},
        readonly: true,
      }
    },
    created() {
      this.param.id = this.$route.query.id;
      this.readonly = (this.$route.query.type === 'edit' || this.$route.query.type === 'add') ? false : true;
    },
    methods: {
      backClick() {
        this.$router.back();
      },
      submit() {

      },
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  .bg {
    /*background: rgb(243, 243, 243);*/
    padding-top: 1%;
    padding-bottom: 2%;
  }

  .card-box {
    width: 95%;
    margin: 25px auto;
    .form-container {
      margin-top: 20px;
    }
    .button-container {
      text-align: right;
      margin-right: 5%;
      margin-top: 3%;
    }
  }
</style>
