<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="addMode.value ? handleAddData() : handleUpdateData()"
          >{{ addMode.value ? '新增' : '更新' }}</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface Item {
  name: string;
  age: number;
}
const addMode = ref(true);
const blockData = ref<Item[]>([]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

// POST
async function handleAddData() {
  console.log('add data');
  try {
    const response = await axios.post(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      {
        name: tempData.value.name,
        age: tempData.value.age,
      }
    );
    console.log('Data Added:', response.data);
    fetchData();
  } catch (err) {
    error.value = (err as Error).message;
  }
}

// PATCH
async function handleUpdateData() {
  try {
    const response = await axios.patch(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      {
        name: tempData.value.name,
        age: tempData.value.age,
      }
    );
    console.log('Data Modified:', response.data);
    fetchData();
  } catch (err) {
    error.value = (err as Error).message;
  }
}

// PATCH、DELETE
async function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    addMode.value = false;
    tempData.value.name = data.name;
    tempData.value.age = data.age;
    console.log(addMode.value);
  }
  if (btn.status === 'delete') {
    try {
      const response = await axios.delete<Item[]>(
        `https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`
      );
      console.log('Data DELETE:', data.id);
      fetchData();
    } catch (err) {
      error.value = err.message;
    }
  }
}

const loading = ref(true);
const error = ref<string | null>(null);

// GET
async function fetchData() {
  try {
    const response = await axios.get<Item[]>(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    blockData.value = response.data;
  } catch (err) {
    error.value = err.message;
  }
}

onMounted(function () {
  fetchData();
  console.log(addMode.value);
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
