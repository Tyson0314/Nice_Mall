<template>
<div>
  <div style="padding: 20px;">
    <el-breadcrumb separator-class="el-icon-arrow-right" >
      <el-breadcrumb-item >首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
  </div>
  <hr>
  <div class="detail-container">
    <el-card shadow="never">
      <span class="font-title-medium">商品数据列表</span>
      <el-table
        border
        class="standard-margin"
        ref="goodsTable"
        :data="goodsList"
      >
        <el-table-column label="商品图片" width="120" align="center">
          <template slot-scope="scope">
            <img style="height:80px" :src="scope.row.imageMain">
          </template>
        </el-table-column>
        <el-table-column label="货号" width="100" align="center">
          <template slot-scope="scope">
            <span class="font-small">NO.{{scope.row.goodsId}}</span>
          </template>
        </el-table-column>
        <el-table-column label="商品名称" width="160" align="center">
          <template slot-scope="scope">
            <span class="font-small">{{scope.row.goodsName}}</span><br>
          </template>
        </el-table-column>
        <el-table-column label="品牌" width="130" align="center">
          <template slot-scope="scope">
            <span class="font-small">{{scope.row.goodsBrand}}</span>
          </template>
        </el-table-column>
        <el-table-column label="原价" width="100" align="center">
          <template slot-scope="scope">
            <span class="font-small">￥{{scope.row.goodsPrePrice}}</span><br>
          </template>
        </el-table-column>
        <el-table-column label="现价" width="130" align="center">
          <template slot-scope="scope">
            <span class="font-small">￥{{scope.row.goodsCurPrice}}</span><br>
          </template>
        </el-table-column>

        <el-table-column label="标签" width="130" align="center">
          <template slot-scope="scope">{{scope.row.goodsType}}</template>
        </el-table-column>
        <el-table-column label="库存量" width="130" align="center">
          <template slot-scope="scope">￥{{scope.row.storeGoodsNumber}}</template>
        </el-table-column>
        <el-table-column label="推荐" width="80" align="center">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.goodsRecommend"
              active-color="#13ce66"
              inactive-color="rgb(173, 176, 184)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="秒杀" width="130" align="left">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.goodsCrazy"
              active-color="#13ce66"
              inactive-color="rgb(173, 176, 184)">
            </el-switch>
            <!--弹框-->
            <el-popover
              placement="right"
              width="345"
              trigger="click">
              <el-form ref="form" :model="curGoods">
                <el-form-item label="商品现价" class="modify_item">
                  <el-input v-model="curGoods.goodsCurPrice"></el-input>
                </el-form-item>
                <el-form-item label="商品原价" class="modify_item">
                  <el-input v-model="curGoods.goodsPrePrice"></el-input>
                </el-form-item>
                <el-form-item label="商品库存" class="modify_item">
                  <el-input v-model="curGoods.storeGoodsNumber"></el-input>
                </el-form-item>
                <el-form-item label="开始时间" class="modify_item">
                  <el-time-picker placeholder="选择开始时间" type="date"
                                  @change="getTime"
                                  v-model="curGoods.startTime">
                  </el-time-picker>
                </el-form-item>
                <el-form-item label="结束时间" class="modify_item">
                  <el-time-picker placeholder="选择结束时间" type="date"
                                  @change="getTime"
                                  v-model="curGoods.endTime">
                  </el-time-picker>
                </el-form-item>
              </el-form>
              <el-button slot="reference" plain @click="handleGetCurGoods(scope.row.goodsId)" class="modifyCrazy"
                         v-show="scope.row.goodsCrazy === true">修改</el-button>
            </el-popover>
          </template>

        </el-table-column>
      </el-table>
      <div v-show="hasGoods" style="padding: 20px 0 ; text-align: center">!暂无商品</div>
      <el-button
        class="btn-add"
        @click="handleSubmit"
        size="mini"
        type="primary" plain>
        确认提交
      </el-button>
    </el-card>
  </div>
</div>

</template>
<script>
    export default {
        inject:['reload'],
        data() {
            return {
                goodsList:[],
                modifyGoodsList:[], //TODO 修改之后的疯抢商品信息
                getCurList:[],  //TODO 请求当前疯抢请求返回所有的商品集合
                curGoods:{}, //TODO 请求当前疯抢修改的商品
                hasGoods:false
            }
        },
        created() {
          this.handleGetGoods()
        },
        computed: {
          //计算 修改的按钮
          goodsModifyList(){
            let goodsCrazyList = [];
            this.goodsList.forEach((item)=>{
              if(item.goodsCrazy === true){
                goodsCrazyList.push(parseInt(item.goodsId));
              }
            });
            return goodsCrazyList;
          },
        },
        methods: {
          getTime(val){
            return new Date(val).getTime()
          },
          handleGetGoods(){
            this.$http.get('/buss/getgoods/'+window.localStorage.getItem('userId'), {
              params: {},
              headers: {Authorization: window.localStorage.getItem('token')}
            }).then(res=>{
              if(!res.data.status){
                this.hasGoods = true
              }
              this.goodsList = res.data.data;
            }).catch(err=>{
              console.log(err);
            })
          },
          //TODO 请求 当前商品的信息
          handleGetCurGoods(goodsId){
            this.$http.get('/buss/spike',{
              params:{
                goodsId:goodsId
              },
              headers:{
                Authorization: window.localStorage.getItem('token')
              }
            }).then(res=>{
              this.curGoods = res.data.data;
              this.getCurList.push(this.curGoods);
              let obj = {};
              this.getCurList = this.getCurList.reduce(function(item, next) {
                obj[next.goodsId] ? '' : obj[next.goodsId] = true && item.push(next);
                return item;
              }, []);

              let goodsCrazyList  = this.goodsModifyList;
              console.log(this.goodsModifyList);
              //得到选定的疯抢商品 进行 商品信息的修改
              for(let i=0;i< goodsCrazyList.length;i++){
                for(let j=0;j<this.getCurList.length;j++){
                  if(goodsCrazyList[i] === parseInt(this.getCurList[j].goodsId)){
                    this.modifyGoodsList.push(this.getCurList[j]);
                  }
                }
              }
              // //对象数组去重
              obj = {};
              this.modifyGoodsList = this.modifyGoodsList.reduce(function(item, next) {
                obj[next.goodsId] ? '' : obj[next.goodsId] = true && item.push(next);
                return item;
              }, []);
            }).catch(err=>{
              this.$message.error('此商品请求失败');
            });
          },
          //TODO 得到修改的 id 数组 发送请求
          handleSubmit(){
            this.$http.post('/buss/spike',{
              modifyGoodsList:this.modifyGoodsList
            },{
              headers:{Authorization: window.localStorage.getItem('token')}
            }).then(res=>{
                this.$message.success(res.data.message);
                this.reload();
            }).catch(err=>{
              this.$message.error('修改失败');
            })
          }
        }

    }
</script>
<style>
    .detail-container {
        font-family: ahoma,Helvetica,Arial,'宋体',sans-serif;
        width: 100%;
        margin:30px auto;
        position: relative;
    }
    .standard-margin {
        margin-top: 15px;
    }
    .el-table--scrollable-x .el-table__body-wrapper {
        overflow-x: hidden !important;
    }
    .btn-add{
        margin: 10px 0 !important;
        float: right !important;
    }
    .detail-container  .el-card {
        border: 1px solid #dddee1 !important;
        background-color: #FFF;
        color: #303133;
        -webkit-transition: .3s;
        transition: .3s;
    }
    .modifyCrazy{
      font-size: 13px;
      height: 25px;
      line-height: 25px;
      padding: 0 8px !important;
    }
    .modify_item{
      width: 80%;
      margin: 18px auto;
    }
    .detail-container  .el-form-item__content {
      line-height: 40px;
      position: relative;
      font-size: 14px;
      display: inline-block;
      width: 74%;
    }
    .el-input--suffix .el-input__inner {
      padding-right: 30px;
      width: 205px;
      height: 35px;
      margin: 10px 0;
    }
    .detail-container  .el-input__icon {
      height: 100%;
      width: 25px;
      text-align: center;
      -webkit-transition: all .3s;
      transition: all .3s;
      line-height: 28px;
      position: absolute;
      top: 0 !important;
    }
    .detail-container  .el-date-editor.el-input, .el-date-editor.el-input__inner {
      width: 218px;
    }

</style>


