<template>
    <div>
        <el-row>
            <el-col :span="7" class="pos-order" id="order-list">
                <el-tabs>
                    <el-tab-pane label='点餐'>
                        <el-table :data='tableData' border style='100%'>
                            <el-table-column prop="goodsName" label="商品"></el-table-column>
                            <el-table-column prop="count" label="量" width="60"></el-table-column>
                            <el-table-column prop="price" label="金额" width="70"></el-table-column>
                            <el-table-column label='操作' width='100' fixed='right'>
                                <template scope="scope">
                                    <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                                    <el-button type="text" size="small" @click="addorderList(scope.row)">增加</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                        <div class="totalDiv">
                            <small>数量：</small>
                            <strong>{{totalCount}}</strong>&nbsp&nbsp&nbsp&nbsp
                            <small>总计：</small>
                            <strong>{{totalMoney}}</strong> 元
                        </div>
                        <div class='div-btn'>
                            <el-button type='warning'>挂单</el-button>
                            <el-button type='danger' @click="delAllGoods">清空</el-button>
                            <el-button type='success' @click='checkout'>结账</el-button>
                        </div>
                    </el-tab-pane>

                    <el-tab-pane label='挂单'>
                        挂单
                    </el-tab-pane>

                    <el-tab-pane label='外卖'>
                        外卖
                    </el-tab-pane>
                </el-tabs>
            </el-col>
            <!--商品展示-->
            <el-col :span="17">
                <div class='titleBox'>
                    <span>常用商品</span>
                </div>
                <div class='goods-list'>
                    <ul>
                        <li v-for="goods in oftenGoods" @click="addorderList(goods)">
                            <span>{{goods.goodsName}}</span>
                            <span class="money-color">￥{{goods.price}}元</span>
                        </li>
                    </ul>
                </div>
                <div class="classTab">
                    <el-tabs>
                        <el-tab-pane label='汉堡'>
                            <ul class='cookList'>
                                <li v-for="goods in type0Goods" @click="addorderList(goods)">
                                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                    <span class="foodName">{{goods.goodsName}}</span>
                                    <span class="foodPrice">￥{{goods.price}}元</span>
                                </li>
                            </ul>
                        </el-tab-pane>
                        <el-tab-pane label='小食'>
                            <ul class='cookList'>
                                <li v-for="goods in type1Goods" @click="addorderList(goods)">
                                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                    <span class="foodName">{{goods.goodsName}}</span>
                                    <span class="foodPrice">￥{{goods.price}}元</span>
                                </li>
                            </ul>
                        </el-tab-pane>
                        <el-tab-pane label='饮料'>
                            <ul class='cookList'>
                                <li v-for="goods in type2Goods">
                                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                    <span class="foodName">{{goods.goodsName}}</span>
                                    <span class="foodPrice">￥{{goods.price}}元</span>
                                </li>
                            </ul>
                        </el-tab-pane>
                        <el-tab-pane label='套餐'>
                            <ul class='cookList'>
                                <li v-for="goods in type3Goods" @click="addorderList(goods)">
                                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                    <span class="foodName">{{goods.goodsName}}</span>
                                    <span class="foodPrice">￥{{goods.price}}元</span>
                                </li>
                            </ul>
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
    name: 'pos',
    mounted: function() {
        var orderHeight = document.body.clientHeight;
        document.getElementById("order-list").style.height = orderHeight + 'px';
    },
    data() {
        return {

            totalCount: 0,
            totalMoney: 0,
            //  常用商品
            oftenGoods: [],
            // 商品列表
            type0Goods: [],
            type1Goods: [],
            type2Goods: [],
            type3Goods: [],
            // 已点商品
            tableData: []
        }
    },
    created() {
        axios.post('http://jspang.com/DemoApi/oftenGoods.php')
            .then(response => {
                console.log(response);
                this.oftenGoods = response.data
            })
            .catch(error => {
                console.log(error);
            });

        axios.post('http://jspang.com/DemoApi/typeGoods.php')
            .then(response => {
                console.log(response);
                this.type0Goods = response.data[0]
                this.type1Goods = response.data[1]
                this.type2Goods = response.data[2]
                this.type3Goods = response.data[3]
            })
            .catch(error => {
                console.log(error);
            });
    },
    methods: {
        addorderList(goods) {
            // 先判定 tableData 商品的数组内是否有东西 假定给一个变量存储其状态
            let isHave = false;
            console.log(isHave)
            // 判定这个商品是否存在商品列表
            for (let i = 0; i < this.tableData.length; i++) {
                if (this.tableData[i].goodsId == goods.goodsId) {
                    isHave = true;
                }
            }
            if (isHave) {
                let arr = this.tableData.filter(e => e.goodsId == goods.goodsId)
                arr[0].count++;
            } else {
                let newGoods = { goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1 }
                this.tableData.push(newGoods)
            }
            this.getAllMoney()
        },
        // 删除单个商品
        delSingleGoods(goods) {
            this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId)
           this.getAllMoney()
        },
        // 删除所有商品
        delAllGoods() {
            this.tableData = [];
            this.totalCount = 0;
            this.totalMoney = 0;
        },
        // 模拟结账
        checkout() {
            if (this.totalCount != 0) {
                this.tableData = [];
                this.totalCount = 0;
                this.totalMoney = 0;
                this.$message({
                    message: '结账成功,感谢您为店铺贡献一份力!',
                    type: 'success'
                });
            } else {
              this.$message.error('不能空结,理解您的急切心情');
            }
        },
        // 汇总金额和数量
        getAllMoney() {
            this.totalCount = 0;
            this.totalMoney = 0;
            if (this.tableData) {
                this.tableData.forEach(e => {
                    this.totalCount += e.count;
                    this.totalMoney = this.totalMoney + (e.price * e.count)
                })
            }
        }

    }

}
</script>
<style scoped>
.goods-list {
    border-bottom: 1px solid #C0CCDA;
    height: auto;
    overflow: hidden;
    padding-bottom: 10px;
    background-color: #F9FAFC;
}

.goods-list ul {
    margin: 10px 30px;
}

.goods-list li {
    list-style: none;
    float: left;
    border: 1px solid #E5E9F2;
    padding: 10px;
    margin: 10px;
    background-color: #fff;
    cursor: pointer;
}

.goods-list .money-color {
    color: #58B7FF;
}

.pos {
    font-size: 12px;
}

.pos-order {
    background-color: #F9FAFC;
    border-right: 1px solid #C0CCDA;
}

.div-btn {
    text-align: center;
    margin-top: 10px;
}

.totalDiv {
    height: auot;
    overflow: hidden;
    text-align: right;
    font-size: 16px;
    background-color: #fff;
    border-bottom: 1px solid #E5E9F2;
    padding: 10px;
}

.titleBox {
    font-size: 12px;
    height: 21px;
    border-bottom: 1px solid #D3DCE6;
    background-color: #F9FAFC;
    padding: 10px;
}

.cookList {
    margin: 10px 30px;
}

.cookList li {
    list-style: none;
    width: 23%;
    border: 1px solid #E5E9F2;
    height: auot;
    overflow: hidden;
    background-color: #fff;
    padding: 2px;
    float: left;
    margin: 5px;
    cursor: pointer;
}

.cookList li span {

    display: block;
    float: left;
}

.foodImg {
    width: 40%;
}

.foodName {
    font-size: 18px;
    padding-left: 10px;
    color: brown;
}

.foodPrice {
    font-size: 16px;
    padding-left: 10px;
    padding-top: 10px;
}
</style>

