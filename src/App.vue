<script setup lang="ts">
import { reactive, watch, onMounted } from "vue";
import './components/TrainingPlan.css';
interface PlanItem {
  day: string;
  title: string;
  details: string;
  type: "HIIT" | "CARDIO" | "REST";
}

const BASE_WEEK_PLAN: PlanItem[] = [
  { day: "周一", title: "HIIT 全身循环", details: "波比跳20s/休40s×3，登山跑20s/休40s×3，开合跳20s/休40s×3，俯卧撑跳20s/休40s×3（约25–30分钟）", type: "HIIT" },
  { day: "周二", title: "中低强度有氧", details: "快走/慢跑40分钟（心率≈最大心率60–70%）", type: "CARDIO" },
  { day: "周三", title: "HIIT 下肢+核心", details: "冲刺跑20s/休40s×4，深蹲跳20s/休40s×4，平板支撑30s×3（25–30分钟）", type: "HIIT" },
  { day: "周四", title: "中低强度有氧", details: "动感单车40分钟（中等阻力）", type: "CARDIO" },
  { day: "周五", title: "HIIT 上肢+核心", details: "俯卧撑20s/休40s×4，哑铃推举20s/休40s×4，仰卧交替触膝20s/休40s×4（25–30分钟）", type: "HIIT" },
  { day: "周六", title: "中低强度有氧", details: "慢跑/快走35–40分钟（可分两段）", type: "CARDIO" },
  { day: "周日", title: "休息 / 拉伸", details: "瑜伽或全身拉伸20分钟（放松恢复）", type: "REST" },
];

interface DayProgress {
  done: boolean;
  note: string;
}

interface DataState {
  currentWeek: number;
  progress: DayProgress[][];
}

const STORAGE_KEY = "fatloss-training-tracker-vue";

const state = reactive<DataState>({
  currentWeek: 0,
  progress: Array(4)
    .fill(null)
    .map(() => BASE_WEEK_PLAN.map(() => ({ done: false, note: "" }))),
});

onMounted(() => {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (raw) {
    try {
      const saved = JSON.parse(raw);
      if (saved.currentWeek !== undefined && saved.progress) {
        state.currentWeek = saved.currentWeek;
        state.progress = saved.progress;
      }
    } catch {
      console.warn("读取存档失败，使用默认数据");
    }
  }
});

watch(
  state,
  () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  },
  { deep: true }
);

function toggleDone(idx: number) {
  const item = state.progress[state.currentWeek][idx];
  item.done = !item.done;
}

function resetWeek() {
  if (confirm("确定要重置本周所有打卡吗？")) {
    state.progress[state.currentWeek] = BASE_WEEK_PLAN.map(() => ({
      done: false,
      note: "",
    }));
  }
}

function resetAll() {
  if (confirm("确定要清空所有数据吗？")) {
    state.currentWeek = 0;
    state.progress = Array(4)
      .fill(null)
      .map(() => BASE_WEEK_PLAN.map(() => ({ done: false, note: "" })));
  }
}

function prevWeek() {
  if (state.currentWeek > 0) state.currentWeek--;
}

function nextWeek() {
  if (state.currentWeek < 3) state.currentWeek++;
}
</script>

<template>
  <div class="container">
    <header>
      <h1>减脂训练打卡（4 周循环）</h1>
      <p>
        目标：74kg → 65kg ｜ 方案：每周 3 次 HIIT + 3 次有氧 + 1 天恢复。<br />
        点击按钮完成打卡，状态自动保存到本地浏览器。
      </p>
    </header>

    <div class="flex-gap">
      <button
        @click="prevWeek"
        class="btn btn-primary"
        :class="{ 'btn-disabled': state.currentWeek === 0 }"
        :disabled="state.currentWeek === 0"
      >
        上一周
      </button>
      <button
        @click="nextWeek"
        class="btn btn-primary"
        :class="{ 'btn-disabled': state.currentWeek === 3 }"
        :disabled="state.currentWeek === 3"
      >
        下一周
      </button>
      <div class="week-indicator">第{{ state.currentWeek + 1 }}周</div>
    </div>

    <div class="flex-gap-between">
      <!-- 左侧训练列表 -->
      <div class="left-panel">
        <div
          v-for="(plan, idx) in BASE_WEEK_PLAN"
          :key="plan.day"
          :class="['plan-item', { done: state.progress[state.currentWeek][idx].done }]"
        >
          <div class="plan-header">
            <div>
              <div class="plan-day-type">
                {{ plan.day }} ·
                {{
                  plan.type === "HIIT"
                    ? "HIIT"
                    : plan.type === "CARDIO"
                    ? "有氧"
                    : "恢复"
                }}
              </div>
              <h2 class="plan-title">{{ plan.title }}</h2>
            </div>
            <button
              @click="toggleDone(idx)"
              :class="[
                'btn-checkin',
                state.progress[state.currentWeek][idx].done ? 'done' : 'not-done'
              ]"
              :title="
                state.progress[state.currentWeek][idx].done
                  ? '已完成，点击取消'
                  : '未完成，点击打卡'
              "
            >
              {{ state.progress[state.currentWeek][idx].done ? "✅ 已打卡" : "打卡" }}
            </button>
          </div>
          <p class="plan-details">{{ plan.details }}</p>
        </div>
      </div>

      <!-- 右侧备注及操作 -->
      <div class="right-panel">
        <div
          v-for="(plan, idx) in BASE_WEEK_PLAN"
          :key="plan.day + '-note'"
          class="note-block"
        >
          <label :for="'note-' + idx" class="note-label">
            {{ plan.day }} 备注（可选）
          </label>
          <textarea
            v-model="state.progress[state.currentWeek][idx].note"
            :id="'note-' + idx"
            placeholder="今天感觉如何？心率、时间、配速或完成组数等…"
            rows="4"
            class="note-textarea"
          ></textarea>
        </div>

        <div class="actions">
          <button @click="resetWeek" class="btn-reset-week">重置本周</button>
          <button @click="resetAll" class="btn-reset-all">清空全部数据</button>
        </div>
      </div>
    </div>

    <footer>
      <h2>执行小贴士</h2>
      <ul>
        <li>
          HIIT 建议采用 20s 高强度 / 40s 休息；逐周增加 1–2 组或缩短休息时间。
        </li>
        <li>有氧保持“能说话但不能唱歌”的强度，心率维持最大心率的 60–70%。</li>
        <li>每完成 4 周，可点击“清空全部数据”重新开始下一轮。</li>
      </ul>
    </footer>
  </div>
</template>


