<style>
	
</style>
<template>
<div>
    <header class="mipcms-container-header clearfix">
        <div class="header-group">
            <h4> 蜘蛛抓取分析（百度）</h4>
        </div>
    </header>
    <main class="mipcms-container-body" style="height: calc(100% - 50px)">
          <div class="row user-statistical"  v-cloak>
                <div class="col-lg-3">
                    <section class="mip-box">
                        <section class="mip-box-body">
                              <h3>{{todayUserCount}}</h3>
                              <p>今日新增</p>
                        </section>
                    </section>
                </div>
                <div class="col-lg-3">
                    <section class="mip-box">
                        <section class="mip-box-body">
                              <h3>{{yesterdayUserCount}}</h3>
                              <p>昨日蜘蛛</p>
                        </section>
                    </section>
                </div>
                <div class="col-lg-3">
                    <section class="mip-box">
                        <section class="mip-box-body">
                              <h3>{{weekUserCount}}</h3>
                              <p>一周蜘蛛</p>
                        </section>
                    </section>
                </div>
                <div class="col-lg-3">
                    <section class="mip-box">
                        <section class="mip-box-body">
                              <h3>{{allUserCount}}</h3>
                              <p>蜘蛛总数</p>
                        </section>
                    </section>
                </div>
            </div>
   <div class="mip-box">
        <div class="mip-box-body">
              <div id="main" v-loading="loading" style="height:400px;"></div>
            <!--内容列表-->
            <section class="diy-table-list">
                <div class="diy-table-item diy-table-item-header">
                    <ul class="list-unstyled row">
                        <li class="col-md-1">
                            <Checkbox v-model="itemListSelectStatus"  :disabled='!itemList.length' @on-change='itemListSelectChange'></Checkbox>全选
                        </li>
                        <li class="col-md-2">
                            <span>抓取时间</span>
                        </li>
                        <li class="col-md-2">
                            <span>类型</span>
                        </li>
                        <li class="col-md-2">
                            <span>IP</span>
                        </li>
                        <li class="col-md-5">
                            <span>抓取链接</span>
                        </li>
                    </ul>
                </div>
                <div v-if='itemList.length' class="diy-table-body">
                    <div class="diy-table-item" v-for='item in itemList' >
                         <ul class="list-unstyled row">
                                
                            <li class="col-md-1">
                                <Checkbox v-model="item.itemListSelectStatus"></Checkbox>
                            </li>
                            <li class="col-md-2">
                                <span>{{item.add_time|time}}</span>
                            </li>
                            <li class="col-md-2">
                                <span>{{item.type}}</span>
                            </li>
                            <li class="col-md-2 over-h-e">
                                <span>{{item.ua}}</span>
                            </li>
                            <li class="col-md-5 over-h-e">
                                <span>{{item.pageUrl}}</span>
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="no-block" v-else>
                    <Icon type="ios-filing-outline"></Icon>
                    <p>暂无数据</p>
                </div>
            </section>
            
            <!--分页-->
            <div class="text-right clearfix mt-3">
                <Page :total="pagination.total"  @on-page-size-change='itemPaginationSelect' :page-size-opts='[10,100,500,1000,5000]' show-total show-sizer placement='top' @on-change='itemPaginationClick'></Page>
            </div>
        </div>
    </main>
      
</div>
</template>

<script>
    export default {
     data () {
       return {
             itemName: '{$itemName}',
            itemListSelectStatus: false,
            itemList: [],
            loading: false,
            spiderEchart: '',
            spidersTodayList: {'pc':[],'pcRender':[],'mobile':[],'mobileRender':[]},
            
            todayUserCount: 0,
            yesterdayUserCount: 0,
            weekUserCount: 0,
            allUserCount: 0,
            
            pagination: {
                currentPage: 1,
                limit: 10,
                total: this.total,
            },
       }
     },
     watch: {
            
        },
        mounted() {
             this.getItemList();
            this.getSpidersTodayList('pc');
            this.getSpidersTodayList('pcRender');
            this.getSpidersTodayList('mobile');
            this.getSpidersTodayList('mobileRender');
            this.getCount();
        },
        methods: {
            getCount:function() {
                this.$mip.ajax('{$domain}/{$Think.config.admin}/ApiAdminSpider/getCount',{
                }).then(res =>  {
                    if (res.code == 1) {
                        this.todayUserCount = res.data.todayUserCount;
                        this.yesterdayUserCount = res.data.yesterdayUserCount;
                        this.weekUserCount = res.data.weekUserCount;
                        this.allUserCount = res.data.allUserCount;
                    }
                });
            },
            
            getItemList:function() {
                this.$mip.ajax('{$domain}/{$Think.config.admin}/ApiAdminSpider/spidersSelect',{
                    page: this.pagination.currentPage,
                    limit: this.pagination.limit,
                }).then(res =>  {
                    if (res.code == 1) {
                        var itemList = res.data.spidersList;
                        if (itemList) {
                            for (var i = 0; i < itemList.length; i++) {
                                itemList[i].itemListSelectStatus = false; 
                            }
                            this.itemList = itemList;
                        }
                        this.pagination.total = res.data.total;
                    }
                });
            },

            getSpidersTodayList:function(type) {
                this.loading = true;
                this.$mip.ajax('{$domain}/{$Think.config.admin}/ApiAdminSpider/spidersToday',{
                    type:type,
                }).then(res =>  {
                    if (res.code == 1) {
                        var tempData = res.data.spidersTodayList;
                        this.spidersTodayList[type] = tempData;
                        this.spidersTodayList[type].unshift(0);
                        this.getSpiderEchart();
                    }
                    this.loading = false;
                });
            },


            getSpiderEchart: function(){
                 this.spiderEchart = echarts.init(document.getElementById('main'));
                 option = {
                    title: {
                        text: '今日蜘蛛抓取分析'
                    },
                    tooltip: {
                        trigger: 'axis'
                    },
                    legend: {
                        data:['PC端蜘蛛','PC端渲染蜘蛛','移动端蜘蛛','移动端渲染蜘蛛']
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    toolbox: {
                        feature: {
                            saveAsImage: {}
                        }
                    },
                    xAxis: {
                        type: 'category',
                        boundaryGap: false,
                        data: ['00','01','02','03','04','05','06','07','08','09','10','11','12','13','14','15','16','17','18','19','20','21','22','23','24']
                    },
                    yAxis: {
                        type: 'value'
                    },
                    series: [
                        {
                            name:'PC端蜘蛛',
                            type:'line',
                            stack: '页面数',
                            data:this.spidersTodayList.pc,
                            smooth: true,
                        },
                        {
                            name:'PC端渲染蜘蛛',
                            type:'line',
                            stack: '页面数',
                            data:this.spidersTodayList.pcRender,
                            smooth: true,
                        },
                        {
                            name:'移动端蜘蛛',
                            type:'line',
                            stack: '页面数',
                            data:this.spidersTodayList.mobile,
                            smooth: true,
                        },
                        {
                            name:'移动端渲染蜘蛛',
                            type:'line',
                            stack: '页面数',
                            data:this.spidersTodayList.mobileRender,
                            smooth: true,
                        }
                    ]
                };

                this.spiderEchart.setOption(option);
            },
            //选择全部
            itemListSelectChange() {
                var itemList = this.itemList;
                for (var i = 0; i < itemList.length; i++) {
                    if (this.itemListSelectStatus) {
                        itemList[i].itemListSelectStatus = true;
                    } else {
                        itemList[i].itemListSelectStatus = false;
                    }
                }
                this.itemList = itemList;
            },
            itemPaginationSelect(val) {
                this.pagination.limit = val;
                this.getItemList();
            },
            itemPaginationClick(val) {
                this.pagination.currentPage = val;
                this.getItemList();
            },
        }
    }
</script>
