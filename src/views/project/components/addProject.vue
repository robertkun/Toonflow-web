<template>
  <div class="addProject">
    <a-modal v-model:open="addProjectShow" title="新建项目" @ok="handleOk">
      <div class="data">
        <a-form :model="formState">
          <a-form-item label="项目名称">
            <a-input v-model:value="formState.name" />
          </a-form-item>
          <a-form-item label="小说类型">
            <a-select
              v-model:value="formState.type"
              :options="novelTypeOptions"
              placeholder="请选择小说题材类型"
              show-search
              allow-clear
              option-filter-prop="label" />
          </a-form-item>
          <a-form-item label="时代画风">
            <a-select
              v-model:value="formState.artStyle"
              :options="artStyleOptions"
              placeholder="请选择时代画风（世界观+视觉风格）"
              show-search
              allow-clear
              option-filter-prop="label" />
          </a-form-item>
          <!-- <a-form-item label="年代">
            <a-input v-model:value="formState.era" />
          </a-form-item> -->
          <a-form-item label="影片比例">
            <a-select v-model:value="formState.videoRatio" :options="options" />
          </a-form-item>
          <a-form-item label="小说简介">
            <a-textarea v-model:value="formState.intro" :rows="20" style="width: 100%" :autoSize="{ minRows: 3, maxRows: 10 }"></a-textarea>
          </a-form-item>
        </a-form>
      </div>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import axios from "@/utils/axios";
import { message } from "ant-design-vue";
const addProjectShow = defineModel<boolean>();
interface FormState {
  id: number;
  name: string;
  era: string;
  intro: string;
  type: string;
  artStyle: string;
  videoRatio: string;
  createTime: number;
  userId: number;
}
const formState = ref<FormState>({
  id: 0,
  name: "",
  intro: "",
  type: "",
  artStyle: "",
  era: "",
  videoRatio: "",
  createTime: 0,
  userId: 0,
});
const emit = defineEmits(["getProjects"]);
const options = ref([
  { value: "16:9", label: "16:9" },
  { value: "9:16", label: "9:16" },
  // { value: "1:1", label: "1:1" },
  // { value: "4:3", label: "4:3" },
  // { value: "3:4", label: "3:4" },
  // { value: "21:9", label: "21:9" },
]);

const novelTypeOptions = ref([
  {
    label: "男频主流",
    options: [
      { value: "玄幻", label: "玄幻（升级、宗门、异界）" },
      { value: "仙侠", label: "仙侠（修真、飞升、道统）" },
      { value: "武侠", label: "武侠（江湖门派、恩怨快意）" },
      { value: "科幻", label: "科幻（硬科幻、太空、机甲）" },
      { value: "悬疑推理", label: "悬疑推理（案件、反转、博弈）" },
      { value: "都市异能", label: "都市异能（超能、系统、逆袭）" },
    ],
  },
  {
    label: "女频主流",
    options: [
      { value: "古代言情", label: "古代言情（宫廷、权谋、情感线）" },
      { value: "现代言情", label: "现代言情（都市、职场、情感成长）" },
      { value: "豪门总裁", label: "豪门总裁（都市高甜、商战情感）" },
      { value: "年代文", label: "年代文（时代变迁、生活成长）" },
      { value: "女性成长", label: "女性成长（自我实现、现实议题）" },
      { value: "治愈甜宠", label: "治愈甜宠（日常温暖、轻情绪）" },
    ],
  },
  {
    label: "细分与融合",
    options: [
      { value: "历史传奇", label: "历史传奇（史实改编、群像叙事）" },
      { value: "现实主义", label: "现实主义（社会议题、人物弧光）" },
      { value: "惊悚恐怖", label: "惊悚恐怖（氛围压迫、心理恐惧）" },
      { value: "校园青春", label: "校园青春（成长、友情、初恋）" },
      { value: "冒险奇幻", label: "冒险奇幻（任务线、地图探索）" },
      { value: "无限流", label: "无限流（副本闯关、规则生存）" },
    ],
  },
]);

const artStyleOptions = ref([
  {
    label: "古代东方",
    options: [
      { value: "上古神话-国风工笔CG", label: "上古神话（国风工笔 + 高细节CG）" },
      { value: "先秦汉唐-写意国风", label: "先秦汉唐（写意国风 + 水墨质感）" },
      { value: "宋明市井-古风写实", label: "宋明市井（古风写实 + 电影光影）" },
      { value: "仙侠修真-国风幻想CG", label: "仙侠修真（国风幻想 + 粒子特效）" },
      { value: "武侠江湖-新武侠电影风", label: "武侠江湖（新武侠电影风 + 动态运镜）" },
    ],
  },
  {
    label: "近现代与都市",
    options: [
      { value: "民国旧影-胶片电影风", label: "民国旧影（复古胶片 + 暖褐调）" },
      { value: "80年代-写实纪实风", label: "80年代（纪实写实 + 低饱和）" },
      { value: "现代都市-2.5D影视动漫", label: "现代都市（2.5D影视动漫 + 霓虹夜景）" },
      { value: "都市轻喜-明亮轻动画风", label: "都市轻喜（高明度 + 轻动画风）" },
      { value: "现实题材-电影写实风", label: "现实题材（电影写实 + 自然光比）" },
    ],
  },
  {
    label: "未来与科幻",
    options: [
      { value: "近未来-赛博霓虹风", label: "近未来（赛博霓虹 + 高反差）" },
      { value: "太空史诗-硬科幻电影风", label: "太空史诗（硬科幻 + IMAX质感）" },
      { value: "机甲战争-重工业CG风", label: "机甲战争（重工业CG + 金属磨损）" },
      { value: "末日废土-暗色写实风", label: "末日废土（暗色写实 + 尘雾颗粒）" },
    ],
  },
  {
    label: "奇幻与二次元",
    options: [
      { value: "西方奇幻-史诗油画风", label: "西方奇幻（史诗油画 + 魔法粒子）" },
      { value: "日式热血-2D动画风", label: "日式热血（高线条张力 + 漫画分镜）" },
      { value: "韩漫唯美-柔光渲染风", label: "韩漫唯美（柔光渲染 + 角色美型）" },
      { value: "奇诡悬疑-暗黑哥特风", label: "奇诡悬疑（暗黑哥特 + 冷色调）" },
      { value: "童话幻想-治愈插画风", label: "童话幻想（治愈插画 + 粉彩氛围）" },
    ],
  },
]);

function handleOk() {
  axios
    .post("/project/addProject", {
      name: formState.value.name ? formState.value.name : "名称",
      intro: formState.value.intro ? formState.value.intro : "这个是一条小说简介",
      type: formState.value.type ? formState.value.type : "玄幻",
      artStyle: formState.value.artStyle ? formState.value.artStyle : "动漫",
      videoRatio: formState.value.videoRatio ? formState.value.videoRatio : "16:9",
      // era: formState.value.era ? formState.value.era : "现代",
    })
    .then(({ data }) => {
      message.success(`新增项目成功`);
      emit("getProjects");
    })
    .catch(() => {
      message.error(`新增项目失败`);
    })
    .finally(() => {
      addProjectShow.value = false;
    });
}
</script>

<style lang="scss" scoped></style>
