<template>
  <div id="app">
    <b-container>
      <b-row align-h="center">
        <b-col sm="12" md="6">
          <div class="section-filter">
            <b-button v-b-modal.modal-filter variant="outline-success">필터</b-button>
            <Filters
              :category="category"
              :categoryOptionList="categoryOptionList"
              :handleCategoryFilter="handleCategoryFilter"
              @updateCategory="updateCategory"
            />
            <div class="sort">
              <b-button
                variant="light"
                class="asc"
                v-on:click="handleSortAsc"
                :class="{ selected: ord === 'asc' }"
              >오름차순</b-button>
              <b-button
                variant="light"
                class="desc"
                v-on:click="handleSortDesc"
                :class="{ selected: ord === 'desc' }"
              >내림차순</b-button>
            </div>
          </div>
        </b-col>
      </b-row>
    </b-container>

    <b-container>
      <b-row align-h="center">
        <b-col sm="12" md="6">
          <List
            :items="items"
            :loading="loading"
            :categoryOptionList="categoryOptionList"
            :getDetails="getDetails"
          />
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import axios from "axios";
import qs from "qs";
import debounce from "lodash/debounce";
import Filters from "./components/Filters";
import List from "./components/List";
import { BASE_URL } from "./constant";

axios.defaults.baseURL = BASE_URL;

export default {
  name: "app",
  data() {
    return {
      loading: false,
      itemsCurrentPage: 1,
      itemsLastPage: null,
      adsCurrentPage: 1,
      adsLastPage: null,
      ord: "asc",
      limit: 10,
      items: [],
      ads: [],
      maxIndex: 1,
      category: [1, 2, 3],
      categoryOptionList: [],
      details: {}
    };
  },
  components: {
    Filters,
    List
  },
  created() {
    this.handleDebouncedScroll = debounce(this.handleScroll, 100);
    window.addEventListener("scroll", this.handleDebouncedScroll);
  },
  destroyed() {
    window.removeEventListener("scroll", this.handleDebouncedScroll);
  },
  mounted() {
    this.getAds();
    this.getCategory();
    this.getList();
  },
  methods: {
    updateCategory(category) {
      this.category = category;
    },
    handleScroll() {
      if (
        window.innerHeight + window.scrollY >
        document.body.offsetHeight * 0.8
      ) {
        if (this.itemsCurrentPage < this.itemsLastPage) {
          this.itemsCurrentPage++;
          this.getList();
        }
      }
    },
    handleReset() {
      this.items = [];
      this.ads = [];
      this.maxIndex = 1;
      this.itemsCurrentPage = 1;
      this.itemsLastPage = null;
      this.adsCurrentPage = 1;
      this.adsLastPage = null;
    },
    handleSortAsc() {
      this.ord = "asc";
      this.handleReset();
      this.getAds();
      this.getList();
    },
    handleSortDesc() {
      this.ord = "desc";
      this.handleReset();
      this.getAds();
      this.getList();
    },
    handleCategoryFilter() {
      this.handleReset();
      this.getAds();
      this.getList();
    },
    async getList() {
      this.loading = true;
      await axios
        .get("/api/list", {
          params: {
            page: this.itemsCurrentPage,
            ord: this.ord,
            limit: this.limit,
            category: this.category
          },
          paramsSerializer: params => {
            return qs.stringify(params);
          }
        })
        .then(async res => {
          this.itemsLastPage = res.data.list.last_page;
          let items = res.data.list.data;

          await Promise.all(
            items.map(async item => {
              const details = await this.getDetails(item.id);
              item.email = details.data.info.user.email;
            })
          );

          items.map(item => {
            const category = this.categoryOptionList.find(
              category => category.id === item.category_id
            );
            item.category_name = category.name;
            return item;
          });

          this.items.push(...items);
          if (this.ads.length < this.limit * 0.5) {
            this.adsCurrentPage++;
            await this.getAds();
          }
        })
        .then(() => {
          for (let i = this.maxIndex; i < this.items.length + 1; i++) {
            if (i % 4 === 0 && i !== 0) {
              this.items.splice(i - 1, 0, this.ads.shift());
            }
          }
          this.maxIndex = this.items.length;
          this.loading = false;
        });
    },
    async getCategory() {
      this.loading = true;
      await axios.get("/api/category").then(res => {
        this.categoryOptionList.push(...res.data.list);
        this.loading = false;
      });
    },
    async getAds() {
      this.loading = true;
      await axios
        .get("/api/ads", {
          params: {
            page: this.adsCurrentPage,
            limit: this.limit
          }
        })
        .then(res => {
          this.ads.push(...res.data.list.data);
          this.loading = false;
        });
    },
    getDetails(id) {
      this.loading = true;
      return axios.get("/api/view", {
        params: {
          id
        }
      });
    }
  }
};
</script>
<style lang="less">
@import "./base.less";

.section-filter {
  display: flex;
  background: #fff;
  align-items: center;
  margin: 10px 0;
  .sort {
    margin-left: auto;
    button {
      padding: 10px;
      margin-left: 10px;
      cursor: pointer;
      &.selected {
        color: @main-color;
        font-weight: bold;
      }
    }
  }
}
</style>
