<template>
  <div class="item_list">
    <van-tabs v-model="navActive"
              @click="handleTabClick">
      <van-tab v-for="(nav, index) in navList"
               :title="nav.name"
               :key="index">
        <van-list v-model="loading"
                  :finished="finished"
                  :immediate-check="false"
                  finished-text="没有更多了"
                  @load="getGoodsList">
          <van-grid clickable
                    :column-num="2"
                    :border="false"
                    class="grid">
            <van-grid-item v-for="(item, i) in goodsList"
                       :key="i"
                       :text="item.name"
                       @click="itemClick(item.id)">
              <div class="good_container">
                <img :src="item.picUrl" class="good_picture"/>
                <div class="good_text_container">
                  <div class="good_name"> {{ item.name }}</div>
                  <div class="good_brief"> {{ item.brief }}</div>
                  <div> 
                    <span class="good_retail_price">
                      ￥{{(item.retailPrice+'').indexOf('.') < 0 ? item.retailPrice+'.00' : item.retailPrice }}
                    </span>
                    <span style="font-size:10px;text-decoration: line-through;color: rgb(181, 171, 171);">
                      ￥{{(item.counterPrice+'').indexOf('.') < 0 ? item.counterPrice+'.00' : item.counterPrice}}
                    </span>
                  </div>
                  
                </div>
              </div>
            </van-grid-item>
          </van-grid>
        </van-list>

      </van-tab>
    </van-tabs>
  </div>
</template>

<script>
import { goodsCategory, goodsList } from '@/api/api';
import { Card, List, Tab, Tabs, Grid, GridItem } from 'vant';

export default {
  name: 'Item-list',
  props: {
    itemClass: {
      type: [String, Number],
      default: ''
    }
  },

  data() {
    return {
      categoryId: this.itemClass,
      goodsList: [],
      page: 0,
      limit: 10,
      currentCategory: {},
      navList: [],
      navActive: 0,
      loading: false,
      finished: false
    };
  },

  created() {
    this.init();
  },

  methods: {
    handleTabClick(index) {
      this.categoryId = this.navList[index].id;
      this.$router.replace({
        name: 'category',
        query: { itemClass: this.categoryId }
      });
      this.init();
    },
    init() {
      goodsCategory({ id: this.categoryId }).then(res => {
        this.navList = res.data.data.brotherCategory;
        this.currentCategory = res.data.data.currentCategory;
        // 当id是L1分类id时，这里需要重新设置成L1分类的一个子分类的id
        if (res.data.data.parentCategory.id == this.categoryId) {
          this.categoryId = res.data.data.currentCategory.id;
        }

        for (let i = 0; i < this.navList.length; i++) {
          if (this.navList[i].id == this.categoryId) {
            this.navActive = i;
            break;
          }
        }
        this.page = 0;
        this.goodsList = [];
        this.getGoodsList();
      });
    },
    getGoodsList() {
      this.page++;
      goodsList({
        categoryId: this.categoryId,
        page: this.page,
        limit: this.limit
      }).then(res => {
        console.log(res.data.data);
        this.goodsList.push(...res.data.data.list);
        this.loading = false;
        this.finished = res.data.data.page >= res.data.data.pages;
      });
    },
    itemClick(id) {
      console.log(id);
      // this.$router.push(`/items/detail/${id}`);
    }
  },

  components: {
    [List.name]: List,
    [Card.name]: Card,
    [Tab.name]: Tab,
    [Tabs.name]: Tabs,
    [Grid.name]: Grid,
    [GridItem.name]: GridItem
  }
};
</script>

<style lang="scss" scoped>
.item_list {
  background-color: #fff;
}

.grid {
  padding: 0 3.3%;
}

.good_container {
  width: 100%;
  border-radius: 12px;
  text-align: left;
  border: 1px solid rgba(230,230,230,0.5);
  padding-bottom: 7px;
  margin-bottom: -9px;
  box-shadow: 2px 2px 5px 1px rgba(230,230,230,0.5);
}

.good_picture {
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
  width: 100%;
}

.good_text_container {
  padding: 2px 10px;
  text-align: center;
}

.good_name {
  font-size: 14px;
}

.good_brief {
  font-size:12px;
  color: rgb(155, 153, 153);
}

.good_retail_price {
  font-size:13px;
  color: rgb(237, 130, 130);
  margin-right: 5px;
}

</style>
