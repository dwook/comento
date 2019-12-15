<template>
  <b-modal id="modal-filter" :hide-footer="true">
    <template v-slot:default="{ hide }">
      <div class="modal-body">
        <span v-for="item in categoryOptionList" :key="item.id">
          <input type="checkbox" :id="item.id" :value="item.id" v-model="newCategory" />
          <label :for="item.id">{{ item.name }}</label>
        </span>
      </div>
      <div class="modal-footer">
        <b-button variant="success" @click="updateCategory">저장</b-button>
      </div>
    </template>
  </b-modal>
</template>

<script>
export default {
  props: {
    category: Array,
    categoryOptionList: Array,
    handleCategoryFilter: Function
  },
  data() {
    return {
      newCategory: []
    };
  },
  created() {
    this.newCategory = this.category;
  },
  methods: {
    updateCategory() {
      this.$bvModal.hide("modal-filter");
      this.$emit("updateCategory", this.newCategory);
      this.handleCategoryFilter();
    }
  }
};
</script>

<style lang="less" scoped>
@import "../base.less";

.modal-backdrop {
  background: #000;
  opacity: 0.7;
}

.modal-body {
  text-align: center;
  span {
    padding: 0 10px;
  }
  input[type="checkbox"] {
    margin-right: 5px;
  }
}

.modal-footer {
  text-align: right;
}
</style>
