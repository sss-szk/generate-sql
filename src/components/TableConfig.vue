<template>
  <div class="table-config">
    <!-- テーブル名入力 -->
    <div class="input-group">
      <label class="input-label">テーブル名:</label>
      <input v-model="localTableName" @input="updateTableName" placeholder="テーブル名を入力" class="input-field" />
    </div>

    <!-- カラム名ペースト -->
    <div class="input-group">
      <label class="input-label">カラム名（Excelからペースト）:</label>
      <textarea v-model="columnText" @input="parseColumns" placeholder="カラム名を貼り付け" class="input-field"></textarea>
    </div>

    <!-- カラム設定（横並びで整列） -->
    <div v-for="(col, index) in localColumns" :key="index" class="column-config">
      <span class="column-label">{{ col }} :</span>
      <select v-model="localColumnTypes[index]" @change="updateColumns" class="column-select">
        <option value="INT">INT</option>
        <option value="VARCHAR">VARCHAR</option>
        <option value="DATETIME">DATETIME</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  tableName: String,
  columns: Array,
  columnTypes: Array
});

// 親コンポーネントにデータを送るためのイベント
const emit = defineEmits(['update:tableName', 'update:columns', 'update:columnTypes']);

// 親（App.vue）から受け取った値をローカル変数として管理
const localTableName = ref(props.tableName);
const columnText = ref('');
const localColumns = ref([...props.columns]);
const localColumnTypes = ref([...props.columnTypes]);

// 親コンポーネントへテーブル名を更新
const updateTableName = () => {
  emit('update:tableName', localTableName.value);
};

// カラムを解析して更新
const parseColumns = () => {
  localColumns.value = columnText.value.trim().split(/\t/);
  localColumnTypes.value = localColumns.value.map(() => 'INT'); // デフォルトは INT
  updateColumns();
};

// 親コンポーネントへカラム情報を送信
const updateColumns = () => {
  emit('update:columns', [...localColumns.value]);
  emit('update:columnTypes', [...localColumnTypes.value]);
};

// 親からの変更を監視してローカルデータを更新
watch(() => props.tableName, (newVal) => localTableName.value = newVal);
watch(() => props.columns, (newVal) => localColumns.value = [...newVal]);
watch(() => props.columnTypes, (newVal) => localColumnTypes.value = [...newVal]);
</script>

<style scoped>
/* コンポーネント全体の幅を固定 */
.table-config, .data-input {
  width: 600px; /* 最初から幅を確保 */
  margin-left: 20px; /* 左に寄せる */
}

/* 各入力グループ（ラベルとフィールド） */
.input-group {
  display: flex;
  flex-direction: column;
  gap: 5px; /* ラベルとフィールドの間隔 */
}

/* ラベルの見た目 */
.input-label {
  font-weight: bold;
  font-size: 14px;
  text-align: left; /* 左揃え */
}

/* テキスト入力欄とテキストエリア */
.input-field {
  min-width: 100%;
  max-width: 100%;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  background: #fff;
  color: #000;
}

/* カラム名とデータ型の選択を横並びにする */
.column-config {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  max-width: 600px; /* 固定幅 */
  margin-top: 5px;
}

/* カラム名（テキスト）を固定幅にする */
.column-label {
  width: 200px; /* 幅を広めに確保 */
  text-align: left; /* 左揃え */
  font-weight: bold;
}

/* セレクトボックスを統一 */
.column-select {
  width: 150px; /* 幅を固定 */
}
</style>

