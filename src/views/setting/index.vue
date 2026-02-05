<template>
  <div class="settingContainer">
    <span>设置</span>
    <div class="addModel">
      <a-button style="margin-bottom: 10px" type="primary" ghost @click="addModelBtn">新增模型</a-button>
      <vxe-table :data="tableData">
        <vxe-column field="name" title="名称" width="100"></vxe-column>
        <vxe-column field="manufacturer" title="厂商" width="100"></vxe-column>
        <vxe-column field="type" title="类型" width="150"></vxe-column>
        <vxe-column field="model" title="模型" width="150"></vxe-column>
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
    <div class="model">
      <h1 style="font-size: 18px; margin-top: 10px; display: block">模型列表</h1>
      <div v-for="(item, index) in modelData" :key="index">
        <div class="modelData">
          
        </div>
      </div>
    </div>
    <div class="other">
      <h1 class="otherConfiguration">其他配置</h1>
      <div class="prompt">
        <a-card title="提示词" style="width: 100%">
          <template #extra>
            <a-button type="primary" class="addBtn" @click="promptEditorShow = true">
              <div class="c">
                <i-edit theme="outline" size="14" fill="currentColor" />
                编辑提示词
              </div>
            </a-button>
          </template>
          <div class="promptData">管理和自定义AIAgent的提示词,可修改主Agent、子Agent和系统提示词</div>
        </a-card>
      </div>
      <div class="sql">
        <a-card title="数据库" style="width: 100%">
          <template #extra>
            <span class="dangerBadge">危险区域</span>
          </template>
          <div class="sqlData">
            <p>以下操作不可逆，请谨慎执行</p>
            <div class="dangerActions" style="display: flex; gap: 12px">
              <a-button danger @click="deleteAllData">
                <i-clear theme="outline" size="14" fill="currentColor" />
                清空数据库
              </a-button>
              <a-button danger type="primary" @click="clearDatabase">
                <i-delete theme="outline" size="14" fill="currentColor" />
                删除数据库
              </a-button>
            </div>
          </div>
        </a-card>
      </div>
      <div class="about">
        <a-card title="关于" style="width: 100%">
          <div class="aboutData">
            <div class="aboutInfo">
              <div class="aboutLogo">
                <img src="@/assets/logo.png" alt="ToonFlow Logo" class="logoImg" />
                <div class="appInfo">
                  <h2 class="appName">ToonFlow</h2>
                  <span class="appVersion">v1.0.0</span>
                </div>
              </div>
              <p class="aboutDesc">ToonFlow 是一款开源的 AI 驱动漫画/分镜创作工具，帮助创作者快速生成故事分镜和视频内容。</p>

              <div class="aboutLinks">
                <div class="linkItem">
                  <div class="linkIcon">
                    <i-github theme="outline" size="20" fill="currentColor" />
                  </div>
                  <div class="linkContent" @click="openGitHub">
                    <span class="linkTitle">GitHub 仓库</span>
                    <span class="linkDesc">查看源代码、提交 Issue 或贡献代码</span>
                  </div>
                  <i-right theme="outline" size="16" fill="#9ca3af" />
                </div>

                <div class="linkItem">
                  <div class="linkIcon">
                    <i-code theme="outline" size="20" fill="currentColor" />
                  </div>
                  <div class="linkContent" @click="openGitee">
                    <span class="linkTitle">Gitee 仓库</span>
                    <span class="linkDesc">国内镜像，查看源代码或贡献代码</span>
                  </div>
                  <i-right theme="outline" size="16" fill="#9ca3af" />
                </div>

                <div class="linkItem" @click="checkUpdate">
                  <div class="linkIcon">
                    <i-refresh theme="outline" size="20" fill="currentColor" />
                  </div>
                  <div class="linkContent">
                    <span class="linkTitle">检查更新 (GitHub)</span>
                    <span class="linkDesc">前往 GitHub Releases 查看最新版本</span>
                  </div>
                  <i-right theme="outline" size="16" fill="#9ca3af" />
                </div>

                <div class="linkItem" @click="checkUpdateGitee">
                  <div class="linkIcon">
                    <i-refresh theme="outline" size="20" fill="currentColor" />
                  </div>
                  <div class="linkContent">
                    <span class="linkTitle">检查更新 (Gitee)</span>
                    <span class="linkDesc">前往 Gitee Releases 查看最新版本</span>
                  </div>
                  <i-right theme="outline" size="16" fill="#9ca3af" />
                </div>

                <div class="linkItem" @click="openLicense">
                  <div class="linkIcon">
                    <i-file-text theme="outline" size="20" fill="currentColor" />
                  </div>
                  <div class="linkContent">
                    <span class="linkTitle">开源协议</span>
                    <span class="linkDesc">本项目基于 AGPL-3.0 协议开源</span>
                  </div>
                  <i-right theme="outline" size="16" fill="#9ca3af" />
                </div>
              </div>

              <div class="licenseBadge">
                <i-certificate theme="outline" size="16" fill="#9913FA" />
                <span>AGPL-3.0 License</span>
              </div>
            </div>
          </div>
        </a-card>
      </div>
    </div>
    <newAddModel v-model:modelShow="modelShow" v-model:modelForm="modelForm" :state="state" />
    <PromptEditor v-model="promptEditorShow" />
  </div>
</template>

<script setup lang="ts">
import dayjs from "dayjs";
import axios from "@/utils/axios";
import { message, Modal } from "ant-design-vue";
import newAddModel from "./components/addModel.vue";
import PromptEditor from "./components/promptEditor.vue";
interface RowData {
  name: string;
  type: string;
  model: string;
  baseUrl: string;
  manufacturer: string;
  createTime: string;
  apiKey: string;
}
const promptEditorShow = ref(false);
const visibleMap = reactive<Record<string | number, boolean>>({});
function setVisible(id: string | number, val: boolean) {
  visibleMap[id] = val;
}
const tableData = ref<RowData[]>([
  {
    name: "默认模型",
    type: "聊天模型",
    model: "gpt-3.5-turbo",
    apiKey: "213232323232323",
    baseUrl: "https://api.openai.com/v1",
    manufacturer: "OpenAI",
    createTime: "2024-01-01 12:00:00",
  },
]);

const modelData = ref([
  {
    code: "",
    name: "",
    model: "",
  },
]);

//查询模型列表
async function fetchModelList() {}

//模型表单数据
const modelForm = ref<RowData>({
  name: "",
  type: "",
  model: "",
  apiKey: "",
  baseUrl: "",
  manufacturer: "",
  createTime: "",
});
const state = ref("");
const modelShow = ref(false);
//新增模型
function addModelBtn() {
  state.value = "新增模型";
  modelForm.value = {
    name: "",
    type: "",
    model: "",
    apiKey: "",
    baseUrl: "",
    manufacturer: "",
    createTime: "",
  };
  modelShow.value = true;
}
//编辑模型
function editModelBtn(row: RowData) {
  state.value = "编辑模型";
  modelForm.value = { ...row };
  modelShow.value = true;
}
//删除模型
function delModelBtn(row: RowData) {
  console.log("删除模型", row);
}
const selectedShow = ref(false);

//其他配置
function showDoubleConfirm2(step = 1): Promise<boolean> {
  return new Promise((resolve) => {
    const config =
      step === 1
        ? {
            title: "危险操作！确认要删除所有数据表吗？",
            content: "此操作会删除所有数据表，且不可恢复。请谨慎操作！",
            okText: "继续操作",
          }
        : {
            title: "请再次确认",
            content: "真的要删除所有数据表吗？数据将无法恢复！",
            okText: "确认删除",
          };

    Modal.confirm({
      ...config,
      cancelText: "取消",
      centered: true,
      onOk: async () => {
        if (step === 1) {
          resolve(await showDoubleConfirm2(2));
        } else {
          resolve(true);
        }
      },
      onCancel: () => resolve(false),
    });
  });
}
async function clearDatabase() {
  const confirmed = await showDoubleConfirm2();
  if (!confirmed) {
    message.info("操作已取消");
    return;
  }
  try {
    await axios.post("/other/clearDatabase");
    message.success("所有数据表已删除，请重新打开页面");
  } catch {
    message.error("操作失败，请重试");
  }
}
function showDoubleConfirm(step = 1): Promise<boolean> {
  return new Promise((resolve) => {
    const config =
      step === 1
        ? {
            title: "危险操作！确认要清空所有数据表吗？",
            content: "此操作会删除所有数据表的数据，且不可恢复。请谨慎操作！",
            okText: "继续操作",
          }
        : {
            title: "请再次确认",
            content: "真的要清空所有数据表吗？数据将无法恢复！",
            okText: "确认删除",
          };

    Modal.confirm({
      ...config,
      cancelText: "取消",
      centered: true,
      onOk: async () => {
        if (step === 1) {
          resolve(await showDoubleConfirm(2));
        } else {
          resolve(true);
        }
      },
      onCancel: () => resolve(false),
    });
  });
}

async function deleteAllData() {
  const confirmed = await showDoubleConfirm();
  if (!confirmed) {
    message.info("操作已取消");
    return;
  }
  try {
    await axios.post("/other/deleteAllData");
    message.success("所有数据表已清空");
  } catch {
    message.error("操作失败，请重试");
  }
}
const GITHUB_URL = "https://github.com/HBAI-Ltd/Toonflow-app";
const GITEE_URL = "https://gitee.com/HBAI-Ltd/Toonflow-app";

function openGitHub() {
  window.open(GITHUB_URL, "_blank");
}

function openGitee() {
  window.open(GITEE_URL, "_blank");
}

function checkUpdate() {
  window.open(`${GITHUB_URL}/releases`, "_blank");
}

function checkUpdateGitee() {
  window.open(`${GITEE_URL}/releases`, "_blank");
}

function openLicense() {
  window.open(`${GITHUB_URL}/blob/master/LICENSE`, "_blank");
}
</script>

<style lang="scss" scoped>
.settingContainer {
  padding: 50px;
  height: 100vh;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  span {
    font-size: 30px;
    font-weight: bold;
  }
  .addModel {
    margin-top: 20px;
    flex: 0 0 auto;
  }
  .model {
    .modelData {
    }
  }
  .other {
    margin-top: 16px;
    flex: 1 1 auto;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
    .otherConfiguration {
      font-size: 18px;
      margin-top: 10px;
      display: block;
    }
    .prompt {
      margin-top: 20px;
      .promptText {
        font-size: 18px;
        margin-top: 10px;
        display: block;
      }
    }
    .sql {
      margin-top: 20px;
      .sqlText {
        font-size: 18px;
        margin-top: 10px;
        display: block;
      }
      .dangerBadge {
        font-size: 12px;
        color: #ef4444;
        background: #fee2e2;
        padding: 4px 10px;
        border-radius: 20px;
        margin-left: auto;
      }
      .sqlData {
      }
    }
    .about {
      margin-top: 20px;
      .aboutText {
        font-size: 18px;
        margin-top: 10px;
        display: block;
      }
      .aboutData {
        .aboutInfo {
          display: flex;
          flex-direction: column;
          gap: 20px;
          .aboutLogo {
            display: flex;
            align-items: center;
            gap: 16px;
            .logoImg {
              width: 64px;
              height: 64px;
              border-radius: 16px;
              object-fit: contain;
              background: linear-gradient(135deg, #f5f0ff 0%, #ede9fe 100%);
              padding: 8px;
            }

            .appInfo {
              display: flex;
              flex-direction: column;
              gap: 4px;
              .appName {
                font-size: 24px;
                font-weight: 700;
                color: #1f2937;
                margin: 0;
                background: var(--mainGradient);
                -webkit-background-clip: text;
                -webkit-text-fill-color: transparent;
                background-clip: text;
              }

              .appVersion {
                font-size: 14px;
                color: #6b7280;
                background: #f3f4f6;
                padding: 2px 10px;
                border-radius: 20px;
                width: fit-content;
              }
            }
          }
          .aboutDesc {
            color: #6b7280;
            font-size: 14px;
            line-height: 1.6;
            margin: 0;
          }

          .aboutLinks {
            display: flex;
            flex-direction: column;
            gap: 8px;
            .linkItem {
              display: flex;
              align-items: center;
              gap: 16px;
              padding: 16px;
              background: #f9fafb;
              border-radius: 12px;
              cursor: pointer;
              transition: all 0.2s;
              border: 1px solid transparent;
              .linkIcon {
                width: 40px;
                height: 40px;
                display: flex;
                align-items: center;
                justify-content: center;
                background: linear-gradient(135deg, var(--mainColorLight) 0%, #ede9fe 100%);
                border-radius: 10px;
                color: var(--mainColor);
                flex-shrink: 0;
              }
            }

            .linkItem:hover {
              background: #f3f4f6;
              border-color: var(--mainColor);
              transform: translateX(4px);
            }
            .linkContent {
              flex: 1;
              display: flex;
              flex-direction: column;
              gap: 2px;
              .linkTitle {
                font-size: 14px;
                font-weight: 600;
                color: #1f2937;
              }
              .linkDesc {
                font-size: 12px;
                color: #9ca3af;
              }
            }
          }
          .licenseBadge {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            padding: 10px 16px;
            background: linear-gradient(135deg, var(--mainColorLight) 0%, #ede9fe 100%);
            border-radius: 8px;
            font-size: 13px;
            font-weight: 500;
            color: var(--mainColor);
            width: fit-content;
            border: 1px solid rgba(153, 19, 250, 0.2);
          }
        }
      }
    }
  }
}
</style>
