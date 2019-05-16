<template>
  <aside v-if="fields.length == 0">
    {{ $t('example.warning') }}
  </aside>
  <aside v-else>
    <a v-for="(item, index) in fields" :key="index" @click="onFieldClick($event, item)">
      <span v-if="clientFieldName !== '' && clientFieldName === item">
        <strong>{{ item }} [client id]</strong> /
      </span>
      <span v-else>
        {{ item }} /
      </span>
    </a>
  </aside>
</template>

<script>
export default {
  name: 'Fields',
  props: {
    onFieldClick: {
      type: Function,
      required: true
    },
    fields: {
      type: Array,
      required: false,
      default() {
        return []
      }
    },
    clientFieldName: {
      type: String,
      required: false,
      default() {
        return ''
      }
    }
  }
}
</script>
