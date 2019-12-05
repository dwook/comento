<template>
  <b-modal id="modal-filter" :hide-footer="true">
    <template v-slot:default="{ hide }">
      <div class="modal-body">
        <span v-for="item in categoryOptionList" :key="item.id">
          <input
            type="checkbox"
            :id="item.id"
            :value="item.id"
            v-model="category"
          />
          <label :for="item.id">{{ item.name }}</label>
        </span>
      </div>
      <div class="modal-footer">
        <b-button variant="success" v-on:click="updateCategory" @click="hide()">
          저장
        </b-button>
      </div>
    </template>
  </b-modal>
</template>

<script>
export default {
  name: 'Modal',
  props: {
    category: Array,
    categoryOptionList: Array,
    handleCategoryFilter: Function
  },
  methods: {
    updateCategory() {
      this.$emit('updateCategory', this.category);
      this.handleCategoryFilter();
    }
  }
};
</script>
