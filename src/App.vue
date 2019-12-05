<template>
  <div id="app">
    <b-container>
      <b-row align-h="center">
        <b-col sm="12" md="6">
          <div class="section-filter">
            <b-button v-b-modal.modal-filter variant="outline-success"
              >필터</b-button
            >
            <Modal
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
                >오름차순</b-button
              >
              <b-button
                variant="light"
                class="desc"
                v-on:click="handleSortDesc"
                :class="{ selected: ord === 'desc' }"
                >내림차순</b-button
              >
            </div>
          </div>
        </b-col>
      </b-row>
    </b-container>

    <b-container>
      <b-row align-h="center">
        <b-col sm="12" md="6">
          <ul class="list-group">
            <li class="list-item" v-for="(item, index) in items" :key="index">
              <div class="article" v-if="item.category_id">
                <div class="header">
                  <span>{{ item.category_name }}</span>
                  <span class="num">{{ item.id }}</span>
                </div>
                <div class="body">
                  <div class="details">
                    <span class="email">{{ item.email }}</span>
                    <span class="created_at">{{ item.created_at }}</span>
                  </div>
                  <div class="text">
                    <p class="title">{{ item.title }}</p>
                    <p class="contents">{{ item.contents }}</p>
                  </div>
                </div>
              </div>
              <div v-else class="ad">
                <div>
                  <div class="header">
                    <p>Sponsored</p>
                  </div>
                  <div class="body">
                    <img
                      :src="`https://cdn.comento.kr/assignment/${item.img}`"
                    />
                    <div class="text">
                      <p class="title">{{ item.title }}</p>
                      <p class="contents">{{ item.contents }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </li>
          </ul>
          <div v-if="loading">
            <transition name="fade">
              <b-spinner
                v-show="loading"
                variant="success"
                label="Spinning"
              ></b-spinner>
            </transition>
          </div>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import axios from 'axios';
import qs from 'qs';
import debounce from 'lodash/debounce';
import Modal from './components/Modal.vue';
import { BASE_URL } from './constant';

axios.defaults.baseURL = BASE_URL;

export default {
  name: 'app',
  data: () => {
    return {
      loading: false,
      itemsCurrentPage: 1,
      itemsLastPage: null,
      adsCurrentPage: 1,
      adsLastPage: null,
      ord: 'asc',
      limit: 10,
      items: [],
      ads: [],
      maxIndex: 1,
      category: [1, 2, 3],
      categoryOptionList: []
    };
  },
  components: {
    Modal
  },
  created() {
    this.handleDebouncedScroll = debounce(this.handleScroll, 100);
    window.addEventListener('scroll', this.handleDebouncedScroll);
  },
  destroyed() {
    window.removeEventListener('scroll', this.handleDebouncedScroll);
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
      this.ord = 'asc';
      this.handleReset();
      this.getAds();
      this.getList();
    },
    handleSortDesc() {
      this.ord = 'desc';
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
        .get('/api/list', {
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
          const data = res.data.list.data;
          let items = data.map(item => {
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
      await axios.get('/api/category').then(res => {
        this.categoryOptionList.push(...res.data.list);
        this.loading = false;
      });
    },
    async getAds() {
      this.loading = true;
      await axios
        .get('/api/ads', {
          params: {
            page: this.adsCurrentPage,
            limit: this.limit
          }
        })
        .then(res => {
          this.ads.push(...res.data.list.data);
          this.loading = false;
        });
    }
  }
};
</script>

<style lang="less">
@mobile: ~'only screen and (max-width: 480px)';
@main-color: #00c854;
@sub-color: #e1e4e7;

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #282c30;
  margin-top: 20px;
}

li {
  list-style: none;
}

.section-filter {
  display: flex;
  background: #fff;
  align-items: center;
  margin-bottom: 10px;
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

.list-item {
  border: 1px solid @sub-color;
  margin-bottom: 30px;
  .header {
    padding: 15px;
    text-align: left;
  }
  .body {
    padding: 15px;
    text-align: left;
  }
  p {
    margin: 0;
  }
  .title {
    font-weight: bold;
  }
  .title,
  .contents {
    text-overflow: ellipsis;
    overflow: hidden;
  }
}

.article {
  .header {
    display: flex;
    border-bottom: 1px solid @sub-color;
    .num {
      margin-left: auto;
    }
  }
  p {
    white-space: nowrap;
  }
}

.ad {
  background: lighten(@main-color, 55%);
  .header {
    color: @main-color;
    font-weight: bold;
  }
  .body {
    padding-top: 0;
    &:after {
      display: block;
      content: '';
      clear: both;
    }
    img {
      width: 50%;
      float: left;
    }
    .text {
      width: 50%;
      padding: 0 15px;
      box-sizing: border-box;
      float: left;
      .title {
        line-height: 1.4em;
        height: 2.8em;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        margin-bottom: 15px;
      }
      .contents {
        line-height: 1.4em;
        height: 5.6em;
        display: -webkit-box;
        -webkit-line-clamp: 4;
        -webkit-box-orient: vertical;
      }
    }
  }
}

@media @mobile {
  .ad {
    .body {
      img {
        width: 100%;
        margin-bottom: 15px;
      }
      .text {
        width: 100%;
        .contents {
          -webkit-line-clamp: 2;
          height: 2.8em;
        }
      }
    }
  }
}

.modal-backdrop {
  background: #000;
  opacity: 0.7;
}

.modal-body {
  text-align: center;
  span {
    padding: 0 10px;
  }
  input[type='checkbox'] {
    margin-right: 5px;
  }
}

.modal-footer {
  text-align: right;
}
</style>
