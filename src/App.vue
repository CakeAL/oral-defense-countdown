<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { repeat } from "seemly";
import { darkTheme } from "naive-ui";

const remainSeconds = ref<number>(300);
const setMinutes = ref<number>(5);
const setWarnMinute = ref<number>(1);
const currentState = ref<boolean>(false); // false 暂停 ture 开始
const progressState = ref<string>("success");
const minutesOptions = repeat(61, undefined).map((_, i) => ({
  label: String(i),
  value: i,
}));
const interval = ref<number>(0); // 用来存储计时器
const theme = ref<any | undefined>(darkTheme);

onMounted(() => {
  return () => clearInterval(interval.value);
});

const start = () => {
  window.speechSynthesis.speak(new SpeechSynthesisUtterance("请开始答辩"));
  currentState.value = true;
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
  currentState.value = false;
};

const reset = () => {
  clearInterval(interval.value);
  remainSeconds.value = setMinutes.value * 60;
  progressState.value = "success";
  currentState.value = false;
};

const changeTheme = () => {
  if (theme.value === undefined) {
    theme.value = darkTheme;
  } else {
    theme.value = undefined;
  }
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
  <n-config-provider :theme="theme">
    <n-global-style />
    <div class="container">
      <div class="countdown">
        <span>{{ minutes }}:{{ seconds }}</span>
        <n-progress
          type="line"
          height="50"
          :status="progressState"
          :percentage="percentage"
          :indicator-placement="'inside'"
        />
      </div>
      <div>
        <n-space justify="center">
          <br />
          <n-button
            strong
            secondary
            round
            type="primary"
            @click="start"
            size="large"
            :disabled="currentState"
          >
            开始
          </n-button>
          <n-button
            strong
            secondary
            round
            type="info"
            @click="pause"
            size="large"
            :disabled="!currentState"
          >
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
          <n-button strong secondary round @click="changeTheme" size="large">
            切换亮/暗
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
    </div>
  </n-config-provider>
</template>

<style scoped>
.container {
  display: block;
}

.countdown {
  width: 100vw;
  height: 80vh;
  text-align: center;
}

.countdown span {
  font-family: monospace;
  font-weight: 900;
  line-height: 70vh;
  font-size: 30vw;
}
</style>
