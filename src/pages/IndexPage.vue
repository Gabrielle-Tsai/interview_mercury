<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="searchData" label="查詢" />
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '請填入姓名']"
        />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          v-if="isAdd"
          color="primary"
          class="q-mt-md"
          @click="handleClickOption({ label: '新增', status: 'add' }, tempData)"
          >新增</q-btn
        >
        <q-btn
          v-if="!isAdd"
          color="primary"
          class="q-mt-md"
          @click="
            handleClickOption(
              { label: '更新', status: 'update' },
              tempData,
              tempData.index
            )
          "
        >
          更新
        </q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="filteredRows"
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
                @click="handleClickOption(btn, props.row, tempData.index)"
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
import { ref, computed } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
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

const tempData = ref({
  name: '',
  age: '',
  index: null,
});

const isAdd = ref(true);
const searchData = ref('');

const filteredRows = computed(() => {
  return blockData.value.filter((row) =>
    row.name.toLowerCase().includes(searchData.value.toLowerCase())
  );
});

function handleClickOption(btn, data, index) {
  // ...
  if (data.name === '' || data.age === '') {
    return;
  }

  if (btn.status === 'add') {
    blockData.value.push({
      name: data.name,
      age: data.age,
    });

    tempData.value.name = '';
    tempData.value.age = '';
    tempData.value.index = null;
  } else if (btn.status === 'delete') {
    index = blockData.value.findIndex(
      (item) => item.name === data.name && item.age === data.age
    );
    blockData.value.splice(index, 1);
  } else if (btn.status === 'edit') {
    let originalIndex = blockData.value.findIndex(
      (item) => item.name === data.name && item.age === data.age
    );
    tempData.value.name = data.name;
    tempData.value.age = data.age;
    tempData.value.index = originalIndex;

    isAdd.value = false;
  } else if (btn.status === 'update') {
    blockData.value[index] = {
      name: tempData.value.name,
      age: tempData.value.age,
    };

    tempData.value.name = '';
    tempData.value.age = '';
    tempData.value.index = null;

    isAdd.value = true;
  }
}

function isNumber(string) {
  return !isNaN(parseInt(string) && isFinite(Number(string)));
}
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
