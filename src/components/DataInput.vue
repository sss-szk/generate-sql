<template>
  <div class="data-input">
    <!-- データペースト -->
    <div class="input-group">
      <label class="input-label">データ（Excelからペースト）:</label>
      <textarea v-model="dataText" @input="parseData" placeholder="データを貼り付け" class="input-field"></textarea>
    </div>

    <!-- IDENTITY INSERT チェックボックス -->
    <div class="checkbox-group">
      <label class="input-label" for="identityInsert">IDENTITY INSERTを有効にする : </label>
      <input type="checkbox" id="identityInsert" v-model="identityInsertEnabled">
    </div>

    <!-- 生成された SQL（ラベルとボタンを横並び） -->
    <div class="sql-container">
      <div class="label-and-buttons">
        <label class="input-label">生成されたSQL:</label>
        <div class="button-group">
          <button @click="copyToClipboard">コピー</button>
          <button @click="downloadSQL">ダウンロード</button>
        </div>
      </div>
      <pre class="sql-output">{{ sqlQuery }}</pre>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  tableName: String,
  columns: Array,
  columnTypes: Array
});

const dataText = ref('');
const dataRows = ref([]);
const identityInsertEnabled = ref(false); // IDENTITY INSERT の有効/無効

// データを解析
const parseData = () => {
  dataRows.value = dataText.value.trim().split("\n").map(row => row.split("\t"));
};

// 日付フォーマットを YYYY-MM-DD HH:MI:SS に変換
const formatDate = (dateString) => {
  const date = new Date(dateString);
  if (isNaN(date.getTime())) return 'NULL';

  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');

  return `'${year}-${month}-${day} ${hours}:${minutes}:${seconds}'`;
};

// SQLを生成（computed でリアルタイム更新）
const sqlQuery = computed(() => {
  if (!props.tableName || props.columns.length === 0 || dataRows.value.length === 0) return '';

  const insertStatements = dataRows.value.map(row => {
    const values = row.map((val, index) => {
      if (props.columnTypes[index] === 'VARCHAR') {
        return `N'${val.replace(/'/g, "''")}'`;
      }
      if (props.columnTypes[index] === 'DATETIME') {
        return formatDate(val);
      }
      return val;
    }).join(', ');

    return `INSERT INTO ${props.tableName} (${props.columns.join(', ')}) VALUES (${values});`;
  }).join("\n");

  // IDENTITY INSERT の有効/無効を追加
  if (identityInsertEnabled.value) {
    return `SET IDENTITY_INSERT ${props.tableName} ON;\n` + insertStatements + `\nSET IDENTITY_INSERT ${props.tableName} OFF;`;
  }

  return insertStatements;
});

// SQLをコピー
const copyToClipboard = () => {
  navigator.clipboard.writeText(sqlQuery.value).then(() => {
    alert('SQLをコピーしました！');
  });
};

// SQLをダウンロード
const downloadSQL = () => {
  const blob = new Blob([sqlQuery.value], { type: "text/plain" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "insert.sql";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};
</script>

<style scoped>
/* コンポーネント全体の幅を固定 */
.data-input {
  width: 600px;
  margin-left: 20px;
}

/* 各入力グループ（ラベルとフィールド） */
.input-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

/* ラベルの見た目 */
.input-label {
  font-weight: bold;
  font-size: 14px;
  text-align: left;
}

/* テキストエリア（データ入力） */
.input-field {
  min-width: 100%;
  max-width: 100%;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  background: #fff;
  color: #000;
}

/* チェックボックス */
.checkbox-group {
  display: flex;
  align-items: center;
  gap: 5px;
  margin: 10px 0;
}

/* SQL出力エリア */
.sql-container {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

/* ラベルとボタンを横並び */
.label-and-buttons {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* ボタンのグループ（横並び） */
.button-group {
  display: flex;
  gap: 10px;
}

/* SQL出力用のスタイル */
.sql-output {
  background: #2d2d2d;
  color: whitesmoke;
  padding: 10px;
  white-space: pre-wrap;
  border-radius: 5px;
  font-family: "Courier New", monospace;
  min-height: 80px;
  display: block;
}
</style>
