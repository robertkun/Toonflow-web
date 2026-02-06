<template>
  <div class="modelData">
    <a-modal v-model:open="modelDataShow" title="模型数据" :footer="null" width="90%">
      <div class="data">
        <a-button style="margin-bottom: 10px" type="primary" ghost @click="addModelBtn">新增模型</a-button>
        <a-button style="margin-bottom: 10px; margin-left: 10px" type="primary" ghost @click="confirmConfig">确认配置</a-button>
        <vxe-table ref="tableRef" :data="tableData" :radio-config="{ highlight: true }">
          <vxe-column type="radio" title="选中" width="60"></vxe-column>
          <vxe-column field="name" title="名称" width="100"></vxe-column>
          <vxe-column field="manufacturer" title="厂商" width="100"></vxe-column>
          <vxe-column field="type" title="类型" width="150"></vxe-column>
          <vxe-column field="model" title="模型" width="250"></vxe-column>
          <vxe-column field="baseUrl" title="Base URL"></vxe-column>
          <vxe-column field="apiKey" title="API Key">
            <template #default="{ row }">
              <div class="f">
                <div style="margin-right: 8px; font-size: 15px">
                  {{ visibleMap[row.id] ? row.apiKey : "********" }}
                </div>
                <i-preview-open
                  v-if="!visibleMap[row.id]"
                  theme="outline"
                  size="20"
                  fill="#9b9b9b"
                  style="cursor: pointer"
                  @click="setVisible(row.id, true)" />
                <i-preview-close v-else theme="outline" size="20" fill="#9b9b9b" style="cursor: pointer" @click="setVisible(row.id, false)" />
              </div>
            </template>
          </vxe-column>
          <vxe-column field="createTime" title="创建时间" width="160">
            <template #default="{ row }">
              <div>{{ dayjs(row.createTime).format("YYYY-MM-DD HH:mm:ss") }}</div>
            </template>
          </vxe-column>
          <vxe-column :min-width="40" title="操作" align="center" width="200" show-overflow>
            <template #default="{ row }">
              <el-button type="info" size="small" @click="editModelBtn(row)">编辑</el-button>
              <el-button type="danger" size="small" style="margin-left: 10px" @click="delModelBtn(row)">删除</el-button>
            </template>
          </vxe-column>
        </vxe-table>
      </div>
    </a-modal>
    <newAddModel v-model:modelShow="modelShow" v-model:modelForm="modelForm" :state="state" @fetchModelList="fetchModelList" />
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import dayjs from "dayjs";
import { message, Modal } from "ant-design-vue";
import axios from "@/utils/axios";
import newAddModel from "./addModel.vue";
const modelDataShow = defineModel("modelDataShow", {
  type: Boolean,
  required: true,
});
const configingModel = defineModel<{ id: number; promptsId: number; code: string; model: string; name: string }>("configingModel");
interface RowData {
  id: number;
  name: string;
  type: string;
  model: string;
  baseUrl: string;
  manufacturer: string;
  createTime: number;
  apiKey: string;
}
const tableRef = ref();
const state = ref("");
const modelShow = ref(false);
const tableData = ref<RowData[]>([]);
//模型表单数据
const modelForm = ref<RowData>({
  id: 0,
  name: "",
  type: "",
  model: "",
  apiKey: "",
  baseUrl: "",
  manufacturer: "",
  createTime: 0,
});
//新增模型
function addModelBtn() {
  state.value = "新增模型";
  modelForm.value = {
    id: 0,
    name: "",
    type: "",
    model: "",
    apiKey: "",
    baseUrl: "",
    manufacturer: "",
    createTime: 0,
  };
  modelShow.value = true;
}
const visibleMap = reactive<Record<string | number, boolean>>({});
function setVisible(id: string | number, val: boolean) {
  visibleMap[id] = val;
}
//编辑模型
function editModelBtn(row: RowData) {
  state.value = "编辑模型";
  modelForm.value = { ...row };
  modelShow.value = true;
}
watch(
  () => modelDataShow.value,
  (val) => {
    if (val == true) {
      fetchModelList();
    }
  },
  { deep: true },
);
//查询模型列表
async function fetchModelList() {
  const res = await axios.post("/setting/getSetting");
  tableData.value = res.data;
}

//删除模型
function delModelBtn(row: RowData) {
  axios
    .post("/setting/delModel", { id: row.id })
    .then(() => {
      message.success("项目删除成功");
      fetchModelList();
      emit("modelList");
    })
    .catch(() => {
      message.error("项目删除失败");
    });
}
const emit = defineEmits(["modelList"]);
// 确认配置
async function confirmConfig() {
  const selectedRow = tableRef.value?.getRadioRecord();
  if (!selectedRow) {
    message.warning("请先选择一个模型");
    return;
  }
  try {
    const isEdit = configingModel.value?.id && configingModel.value?.model;
    if (isEdit) {
      await axios.post("/setting/configurationModel", {
        id: configingModel.value?.id,
        promptsId: configingModel.value?.promptsId,
        configId: selectedRow.id,
      });
    } else {
      await axios.post("/setting/configurationModel", {
        promptsId: configingModel.value?.promptsId,
        configId: selectedRow.id,
      });
    }
    message.success("配置成功");
    tableRef.value?.clearRadioRow();
    modelDataShow.value = false;
    emit("modelList");
  } catch {
    message.error("配置失败，请重试");
  }
}
</script>

<style lang="scss" scoped></style>
