<template>
   <div class="pos" style="height:100%">
      <el-row style="height:100%">
         <el-col :span="7" class="pos-order" id="order-list">
            <el-tabs>
               <el-tab-pane label="点餐">
                  <el-table :data="tableData" border style="width:100%">
                     <el-table-column prop="goodsName" label="商品名称"></el-table-column>
                     <el-table-column prop="count" label="数量" width="60"></el-table-column>
                     <el-table-column prop="price" label="单价" width="70"></el-table-column>
                     <el-table-column label="操作" width="100" fixed="right">
                        <template slot-scope="scope">
                           <el-button type="text" size="small" @click="delSingGoods(scope.row)">删除</el-button>
                           <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                        </template> 
                     </el-table-column>
                  </el-table>

                  <div class="totalDiv">
                     <small>数量：</small> {{totalCount}}  &nbsp;&nbsp;&nbsp;  <small>金额：</small> {{totalMoney}} 元
                  </div>
                  <div class="div-btn">
                     <el-button type="warning">挂单</el-button>   
                     <el-button type="danger" @click="delAllGoods">删除</el-button>   
                     <el-button type="success" @click="checkout">结账</el-button>   
                  </div>
               </el-tab-pane>
               <el-tab-pane label="挂单">
                  挂单
               </el-tab-pane>
               <el-tab-pane label="外卖">
                  外卖
               </el-tab-pane>
            </el-tabs>
         </el-col>
         <el-col :span="17">
            <div class="often-goods">
               <div class="title">常用商品</div>
               <div class="often-goods-list">
                  <ul>
                     <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                        <span>{{goods.goodsName}}</span>
                        <span class="o-price">￥{{ goods.price }}元</span>
                     </li>
                  </ul>
               </div>
            </div>

            <div class="goods-type">
               <el-tabs>
                  <el-tab-pane label="汉堡">
                     <div>
                        <ul class='cookList'>
                           <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                              <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                              <span class="foodName">{{goods.goodsName}}</span>
                              <span class="foodPrice">￥{{goods.price}}元</span>
                           </li>
                        </ul>
                     </div>
                  </el-tab-pane>
                  <el-tab-pane label="小食">
                     <div>
                        <ul class='cookList'>
                           <li v-for="goods in type1Goods">
                              <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                              <span class="foodName">{{goods.goodsName}}</span>
                              <span class="foodPrice">￥{{goods.price}}元</span>
                           </li>
                        </ul>
                     </div>
                  </el-tab-pane>
                  <el-tab-pane label="饮料">
                     <div>
                        <ul class='cookList'>
                           <li v-for="goods in type2Goods">
                              <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                              <span class="foodName">{{goods.goodsName}}</span>
                              <span class="foodPrice">￥{{goods.price}}元</span>
                           </li>
                        </ul>
                     </div>
                  </el-tab-pane>
                  <el-tab-pane label="套餐">
                     <div>
                        <ul class='cookList'>
                           <li v-for="goods in type3Goods">
                              <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                              <span class="foodName">{{goods.goodsName}}</span>
                              <span class="foodPrice">￥{{goods.price}}元</span>
                           </li>
                        </ul>
                     </div>
                  </el-tab-pane>
               </el-tabs>
            </div>

         </el-col>
      </el-row>
   </div>
</template>

<script>
import axios from 'axios';

export default {
   name:'pos',
   data(){
      return{
         activeName: 'second',
         tableData: [],
         oftenGoods:[],
         type0Goods:[],
         type1Goods:[],
         type2Goods:[],
         type3Goods:[],
         totalMoney:0,  //总金额
         totalCount:0  //总数量
      }
   },
   created(){  //创建之前  ,此处用异步
      axios.get('http://jspang.com/DemoApi/oftenGoods.php')
      .then(res => {
         this.oftenGoods=res.data;
      })
      .catch(error =>{
         console.log(error)
      })

      axios.get('http://jspang.com/DemoApi/typeGoods.php')
      .then(res => {
         this.type0Goods=res.data[0];
         this.type1Goods=res.data[1];
         this.type2Goods=res.data[2];
         this.type3Goods=res.data[3];
      })
      .catch(error =>{
         console.log(error)
      })

   },
   mounted:() => {  //虚拟DOM加载完后
      // const orderHeight = document.body.clientHeight;
      // document.getElementById('order-list').style.height = orderHeight + 'px';
   },
   methods:{
      addOrderList(goods){  //添加商品fn
         this.totalMoney=0;  //先清零
         this.totalCount=0;

         //判断商品是否已存在与订单列表中
         let isHave = false;
         for(let i=0;i<this.tableData.length;i++){
            if(this.tableData[i].goodsId==goods.goodsId){
               isHave=true;
            }
         }
         //根据判断再决定是否添加
        if(isHave){
           //改变订单列表中的商品
           let arr = this.tableData.filter(o =>o.goodsId == goods.goodsId );
           arr[0].count++;
        }else{
           let newGoods = {goodsId:goods.goodsId, goodsName:goods.goodsName,price:goods.price,count:1}
            this.tableData.push(newGoods);
        }
         //计算总金额和数量
         this.getAllMoney()

      },
      delSingGoods(goods){ //删除订单里的单个商品
         this.tableData=this.tableData.filter(o => o.goodsId != goods.goodsId);
         this.getAllMoney()
      },
      //订单全部删除
      delAllGoods(){
         this.tableData=[];
         this.totalCount=0;
         this.totalMoney=0;
      },
      checkout(){ //模拟结账
         if(this.totalCount!=0){
            this.tableData=[];
            this.totalCount=0;
            this.totalMoney=0;
            this.$message({
               message:'结账成功',
               type:'success'
            })
         }else{
            this.$message.error('请先选购商品！')
         }
      },
      //计算总金额和数量
      getAllMoney(){
         this.totalCount=0;
         this.totalMoney=0;
         if(this.tableData){
            this.tableData.forEach(element => {
               this.totalCount+=element.count;
               this.totalMoney=this.totalMoney+(element.price*element.count);
            })
         }
      }
   }
}
</script>

<style>
.pos-order{
   background-color: #F9FAFC;
   border-right:1px solid #C0CCDA;
   height:100%;
}
.div-btn{
   margin-top: 15px;
}
.often-goods{

}
.often-goods .title{
   height:20px;
   border-bottom: 1px solid #D3dce6;
   background-color: #F9FAFC;
   padding:10px;
   text-align: left;
}
.often-goods-list ul li{
   list-style:none;
   float: left;
   border: 1px solid #E5E9F2;
   border-radius: 5px;
   padding:10px;
   margin:10px;
   background-color: #ffffff;
   cursor: pointer;
}
.o-price{
   color:#5887FF;
}
.goods-type{
   clear:both;
   padding:0 10px
}

.cookList li{
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auot;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       float:left;
       margin: 2px;
       cursor:pointer;
 
   }
   .cookList li span{
       
        display: block;
        float:left;
   }
   .foodImg{
       width: 40%;
   }
   .foodName{
       font-size: 16px;
       padding-left: 10px;
       color:brown;
 
   }
   .foodPrice{
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
   }
   .totalDiv{
      background-color: #ffffff;
      padding:10px;
      border-bottom:1px solid #ebeef5;
   }
</style>

