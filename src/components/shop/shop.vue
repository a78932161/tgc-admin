<template>
  <div class="bg">
    <el-card class="card-box">
      <form-container ref="form" :model="form" class="form-container" :_readonly="readonly">
        <field-input label="店名" v-model="form.name" width="2.5" prop="name"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="联系方式" v-model="form.phone" width="2.5" prop="phone"
                     :rules="r(true).all(R.mobile)"></field-input>
        <field-input label="店长名" v-model="form.people" width="2.5" prop="people"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="地址" v-model="form.address" width="2.5" prop="address" disabled="true"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="经度" v-model="form.latitude" width="2.5" prop="latitude" disabled="true"
                     :rules="r(true).all(R.require)"></field-input>
        <field-input label="维度" v-model="form.longitude" width="2.5" prop="longitude" disabled="true"
                     :rules="r(true).all(R.require)"></field-input>
        <!--<field-input label="账号" v-model="user.username" width="2.5"></field-input>-->
        <!--<field-input label="密码" v-model="user.password" width="2.5"></field-input>-->
      </form-container>
      <div style="position: relative">
        <div id="container" style="width: 100%;height: 400px"></div>
        <div class="input-card">
          <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">地址</span></div>
            <input id='address' type="text" value='' @change="geoCode()">
          </div>
          <div class="input-item">
            <div class="input-item-prepend"><span class="input-item-text">经纬度</span></div>
            <input id='lnglat' disabled type="text">
          </div>
        </div>
      </div>

      <div class="button-container">
        <el-button type="primary" @click="backClick">返回</el-button>
        <el-button type="primary" @click="submit" v-show="!readonly">确 定</el-button>
      </div>
    </el-card>
  </div>
</template>


<script>
  import api from 'graph/shop.graphql';
  import {filterAttr} from 'common/js';
  import AMap from 'AMap';

  export default {
    data() {
      return {
        param: {id: ''},
        form: {},
        readonly: true,
        map: '',
        user: {},
      }
    },
    created() {
      this.param.id = this.$route.query.id;
      this.readonly = (this.$route.query.type === 'edit' || this.$route.query.type === 'add') ? false : true;
    },

    apollo: {
      form() {
        return this.getEntityQuery(api.getStore);
      }
    },
    mounted() {
      this.mapStart();
    },
    methods: {
      submit() {
        // this.form.id = this.$route.query.id;
        if (this.$route.query.type == 'add') {
          this.$refs.form.gqlValidate(api.store_add, {
              store: this.form
            }, () => {
              this.$message.success('添加成功');
              this.$router.back();
            }
          )
        } else if (this.$route.query.type == 'edit') {
          this.$refs.form.gqlValidate(api.store_modify, {
            store: this.form
          }, () => {
            this.$message.success('修改成功');
            this.$router.back();
          })
        }
      },
      backClick() {
        this.$router.back();
      },
      mapStart() {
        this.map = new AMap.Map("container", {
          resizeEnable: true
        });
      },
      geoCode() {
        var map = new AMap.Map("container", {
          resizeEnable: true
        });
        var geocoder, marker;
        if (!geocoder) {
          geocoder = new AMap.Geocoder({
            city: "", //城市设为北京，默认：“全国”
          });
        }
        var address = document.getElementById('address').value;

        geocoder.getLocation(address, (status, result) => {
          if (status === 'complete' && result.geocodes.length) {
            var lnglat = result.geocodes[0].location
            document.getElementById('lnglat').value = lnglat;
            this.form['address'] = address;
            this.form['latitude'] = lnglat.lat;
            this.form['longitude'] = lnglat.lng;
            this.$forceUpdate();
            if (!marker) {
              marker = new AMap.Marker();
              map.add(marker);
            }
            marker.setPosition(lnglat);
            map.setFitView(marker);
          } else {
            alert(JSON.stringify(result))
          }
        });
      }


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

  .amap-copyright {
    box-sizing: content-box
  }

  * {
    box-sizing: border-box
  }

  .input-textarea {
    color: grey;
    height: 8em;
    overflow: auto;
    border-radius: 0.4rem;
    border: 1px solid #ced4da;
    margin-bottom: 1rem;
  }

  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
    line-height: 1.5;
    font-weight: 300;
    color: #111213;
  }

  hr {
    margin: 0.5rem 0;
    box-sizing: content-box;
    height: 0;
    overflow: visible;
    border: 0;
    border-top: 1px solid rgba(0, 0, 0, .1)
  }

  p {
    margin-top: 0;
    margin-bottom: 0;
  }

  label {
    display: inline-block;
    margin-bottom: 0.4rem;
  }

  label, .btn {
    margin-left: 0;
    font-size: 1rem;
  }

  button, input, select {
    margin: 0;
    font-family: inherit;
    font-size: inherit;
    line-height: inherit;
    overflow: visible;
    text-transform: none
  }

  [type=button]::-moz-focus-inner, [type=reset]::-moz-focus-inner, [type=submit]::-moz-focus-inner, button::-moz-focus-inner {
    padding: 0;
    border-style: none;
  }

  input[type=checkbox], input[type=radio] {
    box-sizing: border-box;
    padding: 0;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    padding: 0;
    margin: 0 0.5rem 0 0;
  }

  h4 {
    font-family: inherit;
    line-height: 1.8;
    font-weight: 300;
    color: inherit;
    font-size: 1.1rem;
    margin-top: 0;
    margin-bottom: .5rem
  }

  .btn {
    display: inline-block;
    font-weight: 400;
    text-align: center;
    white-space: nowrap;
    vertical-align: middle;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    border: 1px solid transparent;
    transition: color .15s ease-in-out, background-color .15s ease-in-out, border-color .15s ease-in-out, box-shadow .15s ease-in-out;
    background-color: transparent;
    background-image: none;
    color: #25A5F7;
    border-color: #25A5F7;
    padding: .25rem .5rem;
    line-height: 1.5;
    border-radius: 1rem;
    -webkit-appearance: button;
    cursor: pointer;
  }

  .btn:hover {
    color: #fff;
    background-color: #25A5F7;
    border-color: #25A5F7
  }

  .btn:hover {
    text-decoration: none
  }

  .input-item {
    position: relative;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-wrap: wrap;
    flex-wrap: wrap;
    -ms-flex-align: center;
    align-items: center;
    width: 100%;
    height: 3rem;
  }

  .input-item:last-child {
    margin-bottom: 0;
  }

  .input-item > select, .input-item > input[type=text], .input-item > input[type=date] {
    position: relative;
    -ms-flex: 1 1 auto;
    flex: 1 1 auto;
    width: 1%;
    margin-bottom: 0;
  }

  .input-item > select:not(:last-child), .input-item > input[type=text]:not(:last-child), .input-item > input[type=date]:not(:last-child) {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0
  }

  .input-item > select:not(:first-child), .input-item > input[type=text]:not(:first-child), .input-item > input[type=date]:not(:first-child) {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0
  }

  .input-item-prepend {
    margin-right: -1px;
  }

  .input-item-text, input[type=text], input[type=date], select {
    height: calc(2.2rem + 2px);
  }

  .input-item-text {
    width: 6rem;
    text-align: justify;
    padding: 0.4rem 0.7rem;
    display: inline-block;
    text-justify: distribute-all-lines;
    /*ie6-8*/
    text-align-last: justify;
    /* ie9*/
    -moz-text-align-last: justify;
    /*ff*/
    -webkit-text-align-last: justify;
    /*chrome 20+*/
    -ms-flex-align: center;
    align-items: center;
    margin-bottom: 0;
    font-size: 1rem;
    font-weight: 400;
    line-height: 1.5;
    color: #495057;
    text-align: center;
    white-space: nowrap;
    background-color: #e9ecef;
    border: 1px solid #ced4da;
    border-radius: .25rem;
    border-bottom-right-radius: 0;
    border-top-right-radius: 0;
  }

  .input-item-text input[type=checkbox], .input-item-text input[type=radio] {
    margin-top: 0
  }

  .input-card {
    display: flex;
    flex-direction: column;
    min-width: 0;
    word-wrap: break-word;
    background-color: #fff;
    background-clip: border-box;
    border-radius: .25rem;
    width: 22rem;
    border-width: 0;
    border-radius: 0.4rem;
    box-shadow: 0 2px 6px 0 rgba(114, 124, 245, .5);
    position: absolute;
    bottom: 1rem;
    right: 1rem;
    -ms-flex: 1 1 auto;
    flex: 1 1 auto;
    padding: 0.75rem 1.25rem;
  }

  .input-text {
    line-height: 2rem;
    margin-right: 2rem;
  }

  .info hr {
    margin-right: 0;
    margin-left: 0;
    border-top-color: grey;
  }

  .info {
    padding: .75rem 1.25rem;
    margin-bottom: 1rem;
    border-radius: .25rem;
    position: fixed;
    top: 1rem;
    background-color: white;
    width: auto;
    min-width: 22rem;
    border-width: 0;
    right: 1rem;
    box-shadow: 0 2px 6px 0 rgba(114, 124, 245, .5);
  }

  .code {
    left: 1.5rem;
    right: 1.5rem;
    top: 1.5rem;
    bottom: 1.5rem;
    overflow: auto;
    margin-bottom: 0rem;
  }

  .code .btn {
    top: 1rem;
    position: absolute;
    right: 1rem;
  }

  .code .result {
    border: 1px solid rgba(0, 0, 0, 0.1);
    border-radius: 0.5rem;
    padding: 1rem;
    bottom: 1rem;
    position: absolute;
    top: 5.5rem;
    right: 1rem;
    left: 1rem;
    overflow: auto;
  }

  .code .status {
    color: #80adff;
    display: inline-block;
    font-size: 14px;
  }

  .code h4 {
    display: inline-block;
    max-width: 20rem;
    margin-right: 1rem;
    margin-bottom: 1rem;
  }

  select, input[type=text], input[type=date] {
    display: inline-block;
    width: 100%;
    padding: .375rem 1.75rem .375rem .75rem;
    line-height: 1.5;
    color: #495057;
    vertical-align: middle;
    background: #fff url("data:image/svg+xml;charset=utf8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 4 5'%3E%3Cpath fill='%23343a40' d='M2 0L0 2h4zm0 5L0 3h4z'/%3E%3C/svg%3E") no-repeat right .75rem center;
    background-size: 8px 10px;
    border: 1px solid #ced4da;
    border-radius: .25rem;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none
  }

  input[type=text], input[type=date] {
    background: #fff;
    padding: .375rem .75rem;
  }

  select:focus, input[type=text]:focus, input[type=date]:focus {
    border-color: #80bdff;
    outline: 0;
    box-shadow: 0 0 0 .1rem rgba(128, 189, 255, .1)
  }

  .btn:focus {
    outline: 0;
    box-shadow: none;
  }

  select:focus::-ms-value, input[type=text]:focus::-ms-value, input[type=date]:focus::-ms-value {
    color: #495057;
    background-color: #fff
  }

  /* native toastr */
  .native-toast {
    position: fixed;
    background-color: rgba(50, 50, 50, .8);
    border-radius: 33px;
    color: white;
    left: 50%;
    text-align: center;
    padding: 6px 12px;
    opacity: 0;
    z-index: 99999;
    transition: transform .25s, opacity .25s, top .25s;
    box-sizing: border-box;
  }

  .native-toast-bottom {
    bottom: 50px;
    -ms-transform: translateX(-50%) translateY(50px);
    transform: translateX(-50%) translateY(50px)
  }

  .native-toast-bottom.native-toast-shown {
    opacity: 1;
    -ms-transform: translateX(-50%) translateY(0);
    transform: translateX(-50%) translateY(0);
  }

  .native-toast-bottom.native-toast-edge {
    bottom: 0;
  }

  .native-toast-top {
    top: 50px;
    -ms-transform: translateX(-50%) translateY(-50px);
    transform: translateX(-50%) translateY(-50px)
  }

  .native-toast-top.native-toast-shown {
    opacity: 1;
    -ms-transform: translateX(-50%) translateY(0);
    transform: translateX(-50%) translateY(0);
  }

  .native-toast-top.native-toast-edge {
    top: 0;
  }

  .native-toast-center {
    top: 0;
    -ms-transform: translateX(-50%) translateY(-50px);
    transform: translateX(-50%) translateY(-50px)
  }

  .native-toast-center.native-toast-shown {
    opacity: 1;
    top: 50%;
    -ms-transform: translateX(-50%) translateY(-50%);
    transform: translateX(-50%) translateY(-50%);
  }

  .native-toast-edge {
    border-radius: 0;
    width: 100%;
    text-align: left;
  }

  @media screen and (min-width: 40rem) {
    .native-toast:not(.native-toast-edge) {
      max-width: 18rem;
    }
  }

  .native-toast-error {
    background-color: #d92727;
    color: white;
  }

  .native-toast-success {
    background-color: #62a465;
    color: white;
  }

  .native-toast-warning {
    background-color: #fdaf17;
    color: white;
  }

  .native-toast-info {
    background-color: #5060ba;
    color: white;
  }

  [class^="native-toast-icon-"] {
    vertical-align: middle;
    margin-right: 8px
  }

  [class^="native-toast-icon-"] svg {
    width: 16px;
    height: 16px;
  }


</style>








