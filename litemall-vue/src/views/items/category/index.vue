<template>
  <div class="item_list">
    <van-tabs v-model="navActive"
              @click="handleTabClick">
      <van-tab v-for="(nav, index) in navList"
               :title="nav.name"
               :key="index">
          <van-grid clickable
                  :column-num="2"
                  :border="false"
                  class="grid">
            <van-grid-item v-for="(item, i) in goodsList"
                       :key="i"
                       :text="item.name"
                       @click="itemClick(item.id)"
                       @grid-item-content-background-color="gray">
              <div class="good_container">
                <img :src="item.picUrl" class="good_picture"/>
                <div class="good_name"> {{ item.name }}</div>
                <div class="good_brief"> {{ item.brief }}</div>
              </div>
            </van-grid-item>
          </van-grid>
        
        <!-- <van-list v-model="loading"
                  :finished="finished"
                  :immediate-check="false"
                  finished-text="没有更多了"
                  @load="getGoodsList">
          <div class="h">
            <div class="name">{{currentCategory.name}}</div>
            <div class="desc">{{currentCategory.desc}}</div>
          </div>
          <van-card v-for="(item, i) in goodsList"
                    :key="i"
                    :desc="item.brief"
                    :title="item.name"
                    :thumb="item.picUrl"
                    :price="item.retailPrice"
                    :origin-price="item.counterPrice"
                    @click="itemClick(item.id)" />
        </van-list> -->

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
  padding: 0 4%;
}

.good_container {
  width: 100%;
  border-radius: 12px;
  text-align: center;
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

.good_name {
  font-size: 14px;
  margin: 1px 5px;
}

.good_brief {
  font-size:12px;
  color: gray;
  margin: 0 5px;
}

</style>
