<template>
  <div class="random">
    <div class="select">
      <a-row type="flex" align="center">
        <a-col style="margin-right: 8px">抽题范围: </a-col>
        <a-col flex="auto">
          <a-tree-select v-model:value="treeValue" :tree-data="treeData" style="width: 100%" tree-checkable allow-clear
            :show-checked-strategy="TreeSelect.SHOW_PARENT" placeholder="Please select" tree-node-filter-prop="label"
            :tree-line="true" />
        </a-col>
      </a-row>
    </div>
    <div class="question">
      <a-spin class="" size="large" :spinning="isLoading">
        <h1 :style="{ 'font-size': fontSize + 'px' }">{{ question }}</h1>
      </a-spin>
    </div>
    <div class="bottom">
      <fontResize v-model:value="fontSize" />
      <div class="button-container">
        <a-button class="rand-btn" type="primary" size="large" @click="randQues()" ref="randButton">
          再来一道
        </a-button>
        <div class="shortcut-hint">快捷键：N</div>
      </div>
    </div>
  </div>

</template>

<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, watch } from 'vue'
import type { TreeSelectProps } from 'ant-design-vue'
import { TreeSelect, message } from 'ant-design-vue'
import { storeToRefs } from 'pinia'

import { randomOneQuestionByPathList } from '../core/random'
import {
  getTreeDataByQues,
  getPathListByTreeValueList,
} from '../core/questions'
import fontResize from '../components/fontResize.vue'
import { globalStore } from '../questions/globalStore.ts'

const randButton = ref<InstanceType<typeof import('ant-design-vue')['Button']> | null>(null);

const store = globalStore();

// 显示的问题
const question = ref('')
// 是否处于加载中
const isLoading = ref<boolean>(false)
// 选择的范围
const treeValue = ref<any>(storeToRefs(store).savePathList_treeValue)
// 选择的字体大小
const fontSize = ref<number>(40)

// 加载选择范围数据
const treeData: TreeSelectProps['treeData'] = getTreeDataByQues()
// 路径列表
var pathList: Array<Array<string | null>> = [[]]

watch(treeValue, () => {
  pathList = getPathListByTreeValueList(treeValue.value)
}, { immediate: true })

const randQues = () => {
  isLoading.value = true
  setTimeout(() => {
    isLoading.value = false
    const r_Question = randomOneQuestionByPathList(pathList)
    if (r_Question) {
      question.value = r_Question
    } else {
      question.value = ''
      message.error('当前抽取范围不存在或没有题目！')
    }
  }, 300)
}

const handleKeyPress = (event: KeyboardEvent) => {
  // 监听 'n' 键
  if (event.key === 'n') {
    // 检查当前焦点元素
    const activeElement = document.activeElement as HTMLElement;
    const isInputFocused = ['INPUT', 'TEXTAREA'].includes(activeElement.tagName) ||
      activeElement.isContentEditable;

    // 仅当焦点不在输入框或其他可编辑元素时触发按钮点击
    if (!isInputFocused) {
      randButton.value?.$el.click(); // 触发按钮点击
    }
  }
};

onMounted(() => {
  randQues()
  window.addEventListener('keyup', handleKeyPress);

  // if (store.savePathList_treeValue) {
  //   message.info("已自动加载上次选择的抽题范围了")
  // }
})

onBeforeUnmount(() => {
  window.removeEventListener('keyup', handleKeyPress);
});
</script>

<style scoped>
.random {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: stretch;
  min-height: var(--content-min-height);
}

.select .ant-row {
  align-items: center;
  margin-bottom: 8px;
}

.random .rand-btn {
  width: 150px;
  height: 60px;
  font-size: 25px;
}

.question {
  align-self: center;
  margin: 0 40px 20px 40px;
}

.question h1 {
  margin: 0;
  cursor: context-menu;
}

.bottom {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.button-container {
  display: flex;
  flex-direction: column;
  /* 垂直排列子元素 */
  align-items: center;
  /* 水平居中对齐 */
}

.shortcut-hint {
  text-align: center;
  margin-top: 6px;
  /* 使提示文字与按钮保持适当的距离 */
  font-size: 12px;
  color: #666666b8;
}
</style>
