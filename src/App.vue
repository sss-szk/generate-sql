<template>
  <div id="container">
    <div id="app">
      <TableConfig v-model:tableName="tableName" v-model:columns="columns" v-model:columnTypes="columnTypes" />
      <DataInput :tableName="tableName" :columns="columns" :columnTypes="columnTypes" />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import TableConfig from './components/TableConfig.vue';
import DataInput from './components/DataInput.vue';

// リアクティブなデータを作成
const tableName = ref('');
const columns = ref([]);
const columnTypes = ref([]);
</script>

<style scoped>
/* 全体のコンテナ */
#container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 800px;
  margin: 0px;
}

#app {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
  align-items: flex-start;
  padding: 0px;
}

</style>


