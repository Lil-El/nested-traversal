<template>
  <div>
    <h2 v-if="outerData">{{ outerData.id }} - {{ outerData.name }}</h2>
    <div v-if="innerData">{{ innerData.id }} - {{ innerData.name }}</div>
    <button @click="handleClick(state.status)">{{ state.status }}</button>
  </div>
</template>

<script setup>
import { computed, reactive, ref } from "vue";

const STATUS = {
  STOP: "STOP",
  RUNNING: "RUNNING",
  FINISH: "FINISH",
};

const data = [
  {
    name: "qxz",
    id: 1,
    children: [ // children也是异步获取
      { id: "1-1", name: "point 1", data: null }, // TODO: data需要异步获取
      { id: "1-2", name: "point 2" },
      { id: "1-3", name: "point 3" },
      { id: "1-4", name: "point 4" },
      { id: "1-5", name: "point 5" },
      { id: "1-6", name: "point 6" },
      { id: "1-7", name: "point 7" },
      { id: "1-8", name: "point 8" },
      { id: "1-9", name: "point 9" },
      { id: "1-10", name: "point 20" },
    ],
  },
  {
    name: "td",
    id: 2,
    children: [
      { id: "2-1", name: "point 1" },
      { id: "2-2", name: "point 2" },
      { id: "2-3", name: "point 3" },
      { id: "2-4", name: "point 4" },
      { id: "2-5", name: "point 5" },
      { id: "2-6", name: "point 6" },
      { id: "2-7", name: "point 7" },
      { id: "2-8", name: "point 8" },
      { id: "2-9", name: "point 9" },
      { id: "2-10", name: "point 20" },
    ],
  },
  {
    name: "lf",
    id: 3,
    children: [
      { id: "3-1", name: "point 1" },
      { id: "3-2", name: "point 2" },
      { id: "3-3", name: "point 3" },
      { id: "3-4", name: "point 4" },
      { id: "3-5", name: "point 5" },
      { id: "3-6", name: "point 6" },
      { id: "3-7", name: "point 7" },
      { id: "3-8", name: "point 8" },
      { id: "3-9", name: "point 9" },
      { id: "3-10", name: "point 20" },
    ],
  },
];

const state = reactive({
  status: STATUS.STOP,
});

/**
 * @param {Array<{children: Array}>} data 
 * @description 嵌套遍历，实现遍历暂停和继续
 * @todo 3. 使用composition API封装，可以获取执行时的数据项（响应式的）
 *       2. 使用Promise进行封装，异步获取children等（适合深层次异步获取）
 *       1. 实现任意深度的嵌套遍历（场景：数据已经装配好）
 * @hard 边界条件处理；每一层下标的处理；数据链获取；
 */
// data, finishCb
// begin(onRunCb) onRunCb: 每一项的回调
const running = (data) => {
  /**
   * NONE: 未开始
   * RUNNING: 运行中
   * STOP: 暂停中
   * FINISH: 已结束
   */
  const STATUS_MAP = {
    NONE: "NONE",
    RUNNING: "RUNNING",
    STOP: "STOP",
    FINISH: "FINISH",
  }
  let status = STATUS_MAP.NONE;
  let outerIndex = 0;
  let innerIndex = 0;
  const next = (finishCallback = () => { }) => {
    if (outerIndex > data.length) return void finishCallback && finishCallback();
    handler(innerIndex, () => {
      innerIndex = 0;
      outerIndex++;
      next(finishCallback);
    })
  }
  const handler = (index, callback) => {
    innerIndex = index;
    if (status === STATUS_MAP.STOP) return void 0;
    if (innerIndex > data[outerIndex].children.length) return void callback();
    console.log(outerIndex, innerIndex);
    setTimeout(() => handler(index + 1, callback), 500);
  };
  return {
    begin (onFinish) {
      if (status == "NONE" || status == "STOP") {
        status = STATUS_MAP.RUNNING
        next(onFinish)
      }
    },
    stop (onStop = () => { }) {
      if (status == STATUS_MAP.RUNNING) {
        status = STATUS_MAP.STOP
        return Promise.resolve().then(onStop)
      }
    }
  }
};

const runner = running(data);
const handleClick = (currentStatus) => {
  if (currentStatus == STATUS.STOP) {
    state.status = STATUS.RUNNING;
    runner.begin(() => (state.status = STATUS.FINISH))
  } else if (currentStatus == STATUS.RUNNING) {
    state.status = STATUS.STOP;
    runner.stop();
  }
};

</script>
