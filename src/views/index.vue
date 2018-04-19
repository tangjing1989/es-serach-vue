<style >
.index {
  width: 100%;
  position: absolute;
  top: 0;
  /* bottom: 0; */
  left: 0;
  text-align: center;
}

.demo-auto-complete-item {
  text-align: left;
  padding: 4px 0;
  border-bottom: 1px solid #f6f6f6;
}
.demo-auto-complete-group {
  font-size: 12px;
  padding: 4px 6px;
}
.demo-auto-complete-group span {
  color: #666;
  font-weight: bold;
}
.demo-auto-complete-group a {
  float: right;
}
.demo-auto-complete-count {
  float: right;
  color: #999;
}

.t {
    font-weight: 400;
    font-size: medium;
    margin-bottom: 1px;
}

.content_left {
    width: 840px;
    padding-left: 121px;
    padding-top: 15px;
}
em {
  font-style:normal;
  color:#c00 ;
  }
</style>
<template>
    <div style="word-break: break-all;word-wrap: break-word;" > 
        <div class="content_left">
          <Col span="24">
              <Tabs @on-click="changeType" style="width:540px">
                  <TabPane :label="item.text" :name="item.value" :key="item.value" v-for="item in searchTypes" />
              </Tabs>
              <AutoComplete v-model.trim="searchContent" :data="serachHistory"  :filter-method="filterMethod"  style="width:540px;margin-top:-20px">
                      <div  v-if="searchContent==''" class="demo-auto-complete-item" v-for="item in hotSearch">
                          <div class="demo-auto-complete-group">
                              <span>{{ item.title }}</span>
                          </div>
                          <Option v-for="option in item.children" :value="option.hotWords" :key="option.id">
                              <span class="demo-auto-complete-title">{{ option.hotWords }}</span>
                              <span class="demo-auto-complete-count">{{ option.score }} 人关注</span>
                          </Option>
                      </div>

              </AutoComplete>
              <Button type="primary"  style="margin-top:-20px" size="large" icon="ios-search" @click="query()">Search</Button>
          </Col>
        </div>
        <div class="content_left">
            <Row   style="float:left;width:100%;margin-top:15px" v-for="item in restult" :value="item.id" :key="item.id" >
                <Col span="24">
                    <div style=" width: 540px; ">
                        <h3 class="t">
                          <a href="" v-html="item.title"/>
                        </h3>
                    </div>
                    <div style="width:540px;margin-top:5px" >
                        <div style="float:left; clear: both;margin-right:10px" align="center">
                            <img :src="JSON.parse(item.thumbnails)[0].url"  style="height:75px;width: 121px;"/>
                        </div>
                          <div  style="font-size: 13px;" v-html="item.summary" >
                          </div>
                    </div>
                </Col>
            </Row>
       </div>
        <div class="content_left ">
          <Page :total="dataCount" :page-size="pageSize"   @on-change="queryData"  v-if="dataCount>0"></Page>
        </div>
    </div>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      restult: [],
      searchContent: "",
      // 初始化信息总条数
      dataCount: 0,
      pageSize:10,
      // 每页显示多少条
      searchType: "",
      searchTypes: [
        { text: "网页", value: "" },
        { text: "文章", value: "article" },
        { text: "图片", value: "picset" },
        { text: "视频", value: "video" }
      ],
      hotSearch: [],
      serachHistory:[],
    };
  },
  methods: {
    changeType(value) {
      this.searchType = value;
    },
    query() {
      this.queryData(1);
    },
    queryData(pageNum) {
      if (!this.searchContent) {
        this.$Message.info({
          content: "请输入查询内容~",
          duration: 2
        });
        return;
      }
      var params = new URLSearchParams;
      params.append("searchContent", this.searchContent);
      params.append("pageNum", pageNum-1);
      params.append("pageSiz", this.pageSize);
      params.append("infoType", this.searchType);
      axios
        .create({
          baseURL: "http://localhost:8050",
          timeout: 30000
        })
        .post("/articles/search", params)
        .then(res => {
          if (res.data.code == 0) {
            if (res.data.data.content.length==0) {
              this.$Message.info({
                content: '很抱歉，没有找到与“'+this.searchContent+'”相关内容。',
                duration: 5
              });
              this.restult = [];
              this.dataCount = 0;
              return;
            }
            this.restult = res.data.data.content;
            this.dataCount = res.data.data.totalElements;
          } else {
            this.restult = [];
            this.dataCount = 0;
          }
        });
    },
    queryHotKeys() {
      var params=new URLSearchParams;
      params.append("infoType", this.searchType);
      axios
        .create({
          baseURL: "http://localhost:8050",
          timeout: 30000
        })
        .post("hotSearch/getHotSearch",params)
        .then(res => {
          if (res.data.code == 0) {
            var hots = [];
            var parent = new Object();
            parent.title = "热搜榜前十";
            parent.children = [];
            for (let index = 0; index < res.data.data.length; index++) {
              const element = res.data.data[index];
              element.id = index;
              parent.children.push(element);
            }
            hots.push(parent);
            this.hotSearch = hots;
          } else {
            this.restult = [];
          }
        });
    },
    querySerachHistory(){
        axios
        .create({
          baseURL: "http://localhost:8050",
          timeout: 30000
        })
        .post("hotSearch/getHotSearchTotal")
        .then(res => {
         this.serachHistory= res.data.data;
        });
    },
     filterMethod (value, option) {
                return option.toUpperCase().indexOf(value.toUpperCase()) !== -1;
            }
  },
  created() {
    this.queryHotKeys();
    this.querySerachHistory();
    // this.query();
  }
};
</script>