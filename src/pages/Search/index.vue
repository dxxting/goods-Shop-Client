<template>
  <div>
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <li class="with-x" v-if="searchParams.categoryName">{{searchParams.categoryName}}
              <i @click="removeCategoryName">×</i>
              </li>
            <li class="with-x" v-if="searchParams.keyword">{{searchParams.keyword}}
              <i @click='removeKeyword'>×</i>
            </li>
            <li class="with-x" v-if="searchParams.trademark">{{searchParams.trademark.split(':')[1]}}
              <i @click='removeTrademark'>×</i>
            </li>
            <li class="with-x" v-for="(prop,index) in searchParams.props" :key="index">{{prop.split(':')[1]}}
              <i @click='removeProp(index)'>×</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <SearchSelector @searchForTrademark='searchForTrademark' @searchForAttrs='searchForAttrs'/>

        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <ul class="sui-nav">
                <li :class="{active:orderArr[0]==='1'}" @click="setOrder('1')">
                  <a href="javaScript:;">综合
                    <i class='iconfont' :class="orderArr[1]==='desc' ? 'icon-down' : 'icon-up'" v-if="orderArr[0]==='1'"></i>
                  </a>
                </li>
                <li :class="{active:orderArr[0]==='3'}" @click="setOrder('3')">
                  <a href="javaScript:;">销量
                    <i class='iconfont' :class="orderArr[1]==='desc' ? 'icon-down' : 'icon-up'" v-if="orderArr[0]==='3'"></i>
                  </a>
                </li>
                <li>
                  <a href="#">新品</a>
                </li>
                <li>
                  <a href="#">评价</a>
                </li>
                <li :class="{active:orderArr[0]==='2'}" @click="setOrder('2')">
                  <a href="javaScript:;">价格
                    <i class='iconfont' :class="orderArr[1]==='desc' ? 'icon-down' : 'icon-up'" v-if="orderArr[0]==='2'"></i>
                  </a>
                </li>
              </ul>
            </div>
          </div>
          <div class="goods-list">
            <ul class="yui3-g">
              <li class="yui3-u-1-5" v-for="(goods,index) in goodsList" :key="goods.id">
                <div class="list-wrap">
                  <div class="p-img">
                    <a href="item.html" target="_blank"><img :src="goods.defaultImg" /></a>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em>
                      <i>{{goods.price}}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a target="_blank" href="item.html" title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】">{{goods.title}}</a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>2000</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a href="success-cart.html" target="_blank" class="sui-btn btn-bordered btn-danger">加入购物车</a>
                    <a href="javascript:void(0);" class="sui-btn btn-bordered">收藏</a>
                  </div>
                </div>
              </li>
            
            </ul>
          </div>
          <!-- 分页器要做的事情1.显示当前页码,2.显示总条数 3.显示总的页码
          前面三点需要从父组件给子组件传递三个数据 当前页 总条数 每页的数量
          4.连续页数 也是由父组件传递给子组件 一般都是奇数
            -->
            <Pagination
            :currentPageNum="searchParams.pageNo"
            :pageSize="searchParams.pageSize"
            :total="goodsListInfo.total"
            :continueNum='5'
            @changeNum="changeNum"></Pagination>
          
        </div>
      </div>
    </div>
  </div>
</template>

<script>


import { mapGetters, mapState } from 'vuex'
  import SearchSelector from './SearchSelector/SearchSelector'
  export default {
    name: 'Search',
    data(){
      return{
        //使我们用户默认搜索的参数,初始化参数
        //是把用户今后可能出现的所有搜索都收集到对象当中,只不过用户操作相关的全部清空
        //后面用户在搜索的时候,根据不同的操作,只需要修改初始化参数,然后再次发送请求就可以
          searchParams:{
              category1Id: '', // 一级分类ID
              category2Id: '', // 二级分类ID
              category3Id: '', // 三级分类ID
              categoryName: '', // 分类名称
              keyword: '', // 搜索关键字
              props: [], // ["属性ID:属性值:属性名"]示例: ["2:6.0～6.24英寸:屏幕尺寸"]
              trademark: '', // 品牌: "ID:品牌名称"示例: "1:苹果"
              order: '1:desc', // 排序方式 1: 综合,2: 价格 asc: 升序,desc: 降序 示例: "1:desc"
              pageNo: 1, // 页码
              pageSize: 2, // 每页数量                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
          }
      }
    },
    beforeMount(){
      //同步更新数据的时候
      this.handlerParams()
    },
    mounted(){
      this.getGoodsListInfo()
    },
  methods:{
    //获取商品列表信息数据
      getGoodsListInfo(){
        // this.searchParams.pageNo=1
        this.$store.dispatch('getGoodsListInfo',this.searchParams)
      },
      //处理请求参数
      handlerParams(){
          let {categoryName,category1Id,category2Id,category3Id} =this.$route.query
          let {keyword} =this.$route.params

          let searchParams ={
            ...this.searchParams,//拆包这个是data中的后面的数据是获取路由中的参数
            categoryName,
            category1Id,
            category2Id,
            category3Id,
            keyword
          };
          //把最终参数当中的空串和属性去掉
          Object.keys(searchParams).forEach((item)=>{
            if(searchParams[item]==='') {
              delete searchParams[item];
            }
          });
          searchParams.pageNo=1
          this.searchParams =searchParams
      },
      //删除面包屑的类名
      removeCategoryName(){
        this.searchParams.pageNo=1
        //清除数据
        // this.searchParams.categoryName=undefined
        delete this.searchParams.categoryName
        //通过 this.getGoodsListInfo()发请求,直接dispatch去发请求,不会去更改之前的路径,因此我们要手动的去修改路径并且发送
        // this.getGoodsListInfo()
        let location={
          name:'search',
          params:this.$route.params
        }
        // this.$router.push(location)
        this.$router.replace(location) 

      },
      //删除面包屑的关键字
      removeKeyword(){
        this.searchParams.pageNo=1
        delete this.searchParams.categoryName

        this.$bus.$emit('clearKeyword')//传递数据
        let location ={
          name:'search',
          query:this.$route.query
        }
        this.$router.replace(location)//push本身是用来跳转路由 不会发送请求 而dispatch是发送请求的,本身
        //并不能改变路由,我们先通过push跳转路由改变路径,然后下面的watch就会监测到路径的变化,然后在监视当中我们通过dispatch发的请求
      },
      //根据品牌搜索
      searchForTrademark(trademark){
        this.searchParams.pageNo=1
        //获取子组件传递过来的trademark对象,拼接成参数所需要的格式
        let trademarkInfo= `${trademark.tmId}:${trademark.tmName}`
        //修改参数
        this.searchParams.trademark =trademarkInfo
        // this.$set(this.searchParams,'trademark',trademark)
        //发送请求
        this.getGoodsListInfo()
      },
      //删除面包屑列表的品牌
      removeTrademark(){
        this.searchParams.pageNo=1
        //删除列表品牌
        this.searchParams.trademark =undefined
        //发送请求 路径不显示直接发送请求
        this.getGoodsListInfo()

      },
      //根据属性搜索
      searchForAttrs(attr,attrValue){
        this.searchParams.pageNo=1
        let prop =`${attr.attrId}:${attrValue}:${attr.attrName}`

        //some every 
        //some是用来判断数组当中是否有一个和条件一样的,如果有有一个是符合的那就为true
        //如果都不一样为false
        //every是用来判断数组当中是否全部都和条件一样,如果是全部一样才是true如果有一个不一样那位false
        let isRepeat =this.searchParams.props.some(item=>item===prop)
        if(isRepeat) return
        this.searchParams.props.push(prop)
        this.getGoodsListInfo()


      },
      //删除面包屑的属性
      removeProp(index){
        this.searchParams.pageNo=1
        this.searchParams.props.splice(index,1)
        this.getGoodsListInfo()
      },
      setOrder(orderFlag){
        this.searchParams.pageNo=1
        this.orderArr
        let [flag,type] =this.orderArr
        if(orderFlag===flag){
          type=type==='desc' ?  'asc' :'desc'
        }else{
          flag=orderFlag
          type ='desc'
        }
          this.searchParams.order=flag +':' +type
          this.getGoodsListInfo()
        
      },
      //点击分页器的按钮改变页码
      changeNum(page){
          this.searchParams.pageNo =page
          this.getGoodsListInfo()
      }
      
  },
  computed:{
    ...mapGetters(['goodsList']),
    ...mapState({
      goodsListInfo:state=>state.search.goodsListInfo
    }),
      orderArr(){
        return this.searchParams.order.split(':')
      }
  },
  watch:{
    $route:{
      handler(){
        this.handlerParams()
        this.getGoodsListInfo()
      
        }
    }
  },
    components: {
      SearchSelector
    }
  }
</script>

<style lang="less" scoped>
  .main {
    margin: 10px 0;

    .py-container {
      width: 1200px;
      margin: 0 auto;

      .bread {
        margin-bottom: 5px;
        overflow: hidden;

        .sui-breadcrumb {
          padding: 3px 15px;
          margin: 0;
          font-weight: 400;
          border-radius: 3px;
          float: left;

          li {
            display: inline-block;
            line-height: 18px;

            a {
              color: #666;
              text-decoration: none;

              &:hover {
                color: #4cb9fc;
              }
            }
          }
        }

        .sui-tag {
          margin-top: -5px;
          list-style: none;
          font-size: 0;
          line-height: 0;
          padding: 5px 0 0;
          margin-bottom: 18px;
          float: left;

          .with-x {
            font-size: 12px;
            margin: 0 5px 5px 0;
            display: inline-block;
            overflow: hidden;
            color: #000;
            background: #f7f7f7;
            padding: 0 7px;
            height: 20px;
            line-height: 20px;
            border: 1px solid #dedede;
            white-space: nowrap;
            transition: color 400ms;
            cursor: pointer;

            i {
              margin-left: 10px;
              cursor: pointer;
              font: 400 14px tahoma;
              display: inline-block;
              height: 100%;
              vertical-align: middle;
            }

            &:hover {
              color: #28a3ef;
            }
          }
        }
      }

      .details {
        margin-bottom: 5px;

        .sui-navbar {
          overflow: visible;
          margin-bottom: 0;

          .filter {
            min-height: 40px;
            padding-right: 20px;
            background: #fbfbfb;
            border: 1px solid #e2e2e2;
            padding-left: 0;
            border-radius: 0;
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

            .sui-nav {
              position: relative;
              left: 0;
              display: block;
              float: left;
              margin: 0 10px 0 0;

              li {
                float: left;
                line-height: 18px;

                a {
                  display: block;
                  cursor: pointer;
                  padding: 11px 15px;
                  color: #777;
                  text-decoration: none;
                }

                &.active {
                  a {
                    background: #e1251b;
                    color: #fff;
                  }
                }
              }
            }
          }
        }

        .goods-list {
          margin: 20px 0;

          ul {
            display: flex;
            flex-wrap: wrap;

            li {
              height: 100%;
              width: 20%;
              margin-top: 10px;
              line-height: 28px;

              .list-wrap {
                .p-img {
                  padding-left: 15px;
                  width: 215px;
                  height: 255px;

                  a {
                    color: #666;

                    img {
                      max-width: 100%;
                      height: auto;
                      vertical-align: middle;
                    }
                  }
                }

                .price {
                  padding-left: 15px;
                  font-size: 18px;
                  color: #c81623;

                  strong {
                    font-weight: 700;

                    i {
                      margin-left: -5px;
                    }
                  }
                }

                .attr {
                  padding-left: 15px;
                  width: 85%;
                  overflow: hidden;
                  margin-bottom: 8px;
                  min-height: 38px;
                  cursor: pointer;
                  line-height: 1.8;
                  display: -webkit-box;
                  -webkit-box-orient: vertical;
                  -webkit-line-clamp: 2;

                  a {
                    color: #333;
                    text-decoration: none;
                  }
                }

                .commit {
                  padding-left: 15px;
                  height: 22px;
                  font-size: 13px;
                  color: #a7a7a7;

                  span {
                    font-weight: 700;
                    color: #646fb0;
                  }
                }

                .operate {
                  padding: 12px 15px;

                  .sui-btn {
                    display: inline-block;
                    padding: 2px 14px;
                    box-sizing: border-box;
                    margin-bottom: 0;
                    font-size: 12px;
                    line-height: 18px;
                    text-align: center;
                    vertical-align: middle;
                    cursor: pointer;
                    border-radius: 0;
                    background-color: transparent;
                    margin-right: 15px;
                  }

                  .btn-bordered {
                    min-width: 85px;
                    background-color: transparent;
                    border: 1px solid #8c8c8c;
                    color: #8c8c8c;

                    &:hover {
                      border: 1px solid #666;
                      color: #fff !important;
                      background-color: #666;
                      text-decoration: none;
                    }
                  }

                  .btn-danger {
                    border: 1px solid #e1251b;
                    color: #e1251b;

                    &:hover {
                      border: 1px solid #e1251b;
                      background-color: #e1251b;
                      color: white !important;
                      text-decoration: none;
                    }
                  }
                }
              }
            }
          }
        }

        .page {
          width: 733px;
          height: 66px;
          overflow: hidden;
          float: right;

          .sui-pagination {
            margin: 18px 0;

            ul {
              margin-left: 0;
              margin-bottom: 0;
              vertical-align: middle;
              width: 490px;
              float: left;

              li {
                line-height: 18px;
                display: inline-block;

                a {
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  border: 1px solid #e0e9ee;
                  margin-left: -1px;
                  font-size: 14px;
                  padding: 9px 18px;
                  color: #333;
                }

                &.active {
                  a {
                    background-color: #fff;
                    color: #e1251b;
                    border-color: #fff;
                    cursor: default;
                  }
                }

                &.prev {
                  a {
                    background-color: #fafafa;
                  }
                }

                &.disabled {
                  a {
                    color: #999;
                    cursor: default;
                  }
                }

                &.dotted {
                  span {
                    margin-left: -1px;
                    position: relative;
                    float: left;
                    line-height: 18px;
                    text-decoration: none;
                    background-color: #fff;
                    font-size: 14px;
                    border: 0;
                    padding: 9px 18px;
                    color: #333;
                  }
                }

                &.next {
                  a {
                    background-color: #fafafa;
                  }
                }
              }
            }

            div {
              color: #333;
              font-size: 14px;
              float: right;
              width: 241px;
            }
          }
        }
      }
    }
  }
</style>