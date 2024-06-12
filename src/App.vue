<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { repeat } from "seemly";

const remainSeconds = ref<number>(300);
const setMinutes = ref<number>(5);
const setWarnMinute = ref<number>(1);
// const currentState = ref<boolean>(false); // false 暂停 ture 开始
const progressState = ref<string>("success");
const minutesOptions = repeat(61, undefined).map((_, i) => ({
  label: String(i),
  value: i,
}));

const interval = ref<number>(0); // 用来存储计时器

onMounted(() => {
  return () => clearInterval(interval.value);
});

const start = () => {
  window.speechSynthesis.speak(new SpeechSynthesisUtterance("请开始答辩"));
  interval.value = setInterval(() => {
    if (remainSeconds.value <= 1) {
      clearInterval(interval.value);
      progressState.value = "error";
      window.speechSynthesis.speak(
        new SpeechSynthesisUtterance("时间结束，请停止答辩")
      );
    } else if (remainSeconds.value === setWarnMinute.value * 60) {
      // 剩余WarnMinute时发出警告
      progressState.value = "warning";
      window.speechSynthesis.speak(
        new SpeechSynthesisUtterance("剩余" + setWarnMinute.value + "分钟")
      );
    }
    remainSeconds.value -= 1;
  }, 1000);
};

const pause = () => {
  clearInterval(interval.value);
};

const reset = () => {
  clearInterval(interval.value);
  remainSeconds.value = setMinutes.value * 60;
  progressState.value = "success";
};

const handleUpdateSetMinutes = () => {
  reset();
};

const minutes = computed(() => {
  let mins = Math.floor(remainSeconds.value / 60);
  if (mins < 10) {
    return "0" + mins;
  } else return mins;
});
const seconds = computed(() => {
  let secs = remainSeconds.value % 60;
  if (secs < 10) {
    return "0" + secs;
  } else return secs;
});
const percentage = computed(() => {
  return 100 - (remainSeconds.value / setMinutes.value / 60) * 100;
});
</script>

<template>
  <div class="container">
    <div class="countdown">
      <span>{{ minutes }} : {{ seconds }}</span>
      <n-progress
        type="line"
        :status="progressState"
        :percentage="percentage"
        :indicator-placement="'inside'"
      />
    </div>
    <n-space justify="center">
      <n-button
        strong
        secondary
        round
        type="primary"
        @click="start"
        size="large"
      >
        开始
      </n-button>
      <n-button strong secondary round type="info" @click="pause" size="large">
        暂停
      </n-button>
      <n-button
        strong
        secondary
        round
        type="warning"
        @click="reset"
        size="large"
      >
        重置
      </n-button>
      <br />
    </n-space>
    <span>设置倒计时时长（分钟）</span>
    <n-select
      v-model:value="setMinutes"
      :options="minutesOptions"
      @update:value="handleUpdateSetMinutes"
    />
    <span>设置剩余时间提醒（分钟）</span>
    <n-select v-model:value="setWarnMinute" :options="minutesOptions" />
  </div>
</template>

<style scoped>

.countdown {
  width: 100vw;
  height: 80vh;
}

.countdown span {
  font-weight: bold;
  font-size: 30vw;
}
</style>
