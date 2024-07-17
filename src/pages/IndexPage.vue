<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <h5>{{ type }} Form</h5>
        <q-form @submit="addData" ref="formRef">
          <q-input
            v-model="tempData.name"
            label="姓名"
            :rules="[(val) => !!val || 'Name cannot be empty.']"
          />
          <q-input
            v-model.number="tempData.age"
            label="年齡"
            :rules="[
              (val) => !!val || 'Age cannot be empty.',
              (val) =>
                (Number.isInteger(val) && val > 0) ||
                'Age must be a positive integer.',
            ]"
          />
          <q-btn type="submit" color="primary" class="q-mt-md">新增</q-btn>
        </q-form>
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
import { ref, reactive, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const type = ref('Add');
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
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
const tableData = reactive([]);
onMounted(() => {
  fetchData();
});
const tempData = ref({
  name: '',
  age: '',
});
const targetId = ref('');
async function handleClickOption(btn, data) {
  console.log(data, 'what is');
  if (btn.status === 'delete') {
    try {
      const response = await axios.delete(
        `https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`
      );
      console.log(response);
    } catch (error) {
      console.log('deleting error, ', error);
    }
    fetchData();
  } else {
    type.value = 'Edit';
    console.log(data, 'what is dast');
    tempData.value = {
      name: data.name,
      age: data.age,
    };
    targetId.value = data.id;
  }
}
const fetchData = async () => {
  try {
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    const newData = response.data;

    blockData.value = newData;
    console.log(newData, 'this is data');
  } catch (error) {
    console.log('fetching error, ', error);
  }
};

const resetAddForm = () => {
  tempData.value = {
    name: '',
    age: '',
  };
};

const addData = async () => {
  console.log('tadakatsu');
  if (type.value === 'Add') {
    const newData = { ...tempData.value, id: Math.random() };
    blockData.value.push(newData);
    console.log(blockData.value, 'sapene');
    try {
      const response = await axios.post(
        'https://dahua.metcfire.com.tw/api/CRUDTest',
        newData,
        { headers: { 'Content-Type': 'application/json' } }
      );

      console.log(response);
    } catch (error) {
      console.log('post error, ', error);
    }
    resetAddForm();
  } else {
    try {
      const response = await axios.patch(
        `https://dahua.metcfire.com.tw/api/CRUDTest`,
        tempData.value,
        { headers: { 'Content-Type': 'application/json' } }
      );

      console.log(response);
    } catch (error) {
      console.log('post error, ', error);
    }
    resetAddForm();
    type.value = 'Add';
  }
};
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
