<template>
  <el-select v-model="comment_disabled" placeholder="Select">
    <el-option
      v-for="item in tableHeader"
      :key="item"
      :label="item"
      :value="item"
    />
  </el-select>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: 'n/a'
    },
    tableHeader: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  computed: {
    comment_disabled: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    }
  }
}
</script>
