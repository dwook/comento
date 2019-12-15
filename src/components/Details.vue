<template>
  <b-modal id="modal-details" :hide-footer="true">
    <template>
      <div class="body">
        <span class="email">{{ details.user.email }}</span>
        <span class="created_at">{{ details.created_at }}</span>
        <div class="text">
          <p class="title">{{ details.title }}</p>
          <p class="contents">{{ details.contents }}</p>
        </div>
      </div>
      <div class="reply">
        <p class="title">Reply</p>
        <ul>
          <li v-for="(reply, index) in details.reply" :key="index">
            <div>
              <strong>{{reply.user.name}}</strong>
            </div>
            <div>{{reply.contents}}</div>
          </li>
        </ul>
      </div>
    </template>
  </b-modal>
</template>

<script>
export default {
  props: {
    details: Object
  }
};
</script>

<style lang="less" scoped>
@import "../base.less";

.body {
  text-align: left;
  border: 1px solid @sub-color;
  padding: 10px;
  .email,
  .created_at {
    color: #888;
    padding-right: 10px;
  }
  .title {
    font-weight: bold;
  }
  .title,
  .contents {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }
}

.reply {
  margin-top: 20px;
  .title {
    color: #888;
  }
  ul {
    overflow-y: scroll;
    height: 400px;
  }
  li {
    text-align: left;
    font-size: 14px;
    padding: 10px 0;
    border-bottom: 1px solid @sub-color;
  }
}
</style>
