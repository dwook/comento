<template>
  <div>
    <ul class="list-group">
      <li class="list-item" v-for="(item, index) in items" :key="index">
        <div class="article" v-if="item.category_id" @click="onItemClick(item.id)">
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
              <img :src="`https://cdn.comento.kr/assignment/${item.img}`" />
              <div class="text">
                <p class="title">{{ item.title }}</p>
                <p class="contents">{{ item.contents }}</p>
              </div>
            </div>
          </div>
        </div>
      </li>
    </ul>
    <Details :details="details" />
    <div v-if="loading" class="loading">
      <transition name="fade">
        <b-spinner v-show="loading" variant="success" label="Spinning"></b-spinner>
      </transition>
    </div>
  </div>
</template>

<script>
import Details from "./Details";

export default {
  props: {
    items: Array,
    loading: Boolean,
    getDetails: Function
  },
  data() {
    return {
      details: {
        user: {
          email: {}
        }
      }
    };
  },
  components: {
    Details
  },
  methods: {
    async onItemClick(id) {
      this.$bvModal.show("modal-details");
      const details = await this.getDetails(id);
      this.details = details.data.info;
    }
  }
};
</script>

<style lang="less">
@import "../base.less";

.loading {
  text-align: center;
}

.list-item {
  border: 1px solid @sub-color;
  margin-bottom: 30px;
  height: 100%;
  .header {
    padding: 15px;
    text-align: left;
  }
  .body {
    padding: 15px;
    text-align: left;
  }
  .email,
  .created_at {
    color: #888;
    padding-right: 10px;
  }
  .title {
    font-weight: bold;
    margin: 0;
  }
  .title,
  .contents {
    text-overflow: ellipsis;
    overflow: hidden;
  }
}

.article {
  cursor: pointer;
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
      content: "";
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
</style>
