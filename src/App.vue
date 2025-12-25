<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-lg-8">
        <h1 class="text-center mb-5">Interval Running Timer</h1>

        <!-- Setup Form -->
        <div v-if="!isRunning" class="card p-4 shadow">
          <div class="row g-4">
            <!-- Active Distance -->
            <div class="col-md-6">
              <label class="form-label fw-bold">Active Distance</label>
              <div class="input-group">
                <input
                  type="number"
                  v-model="activeDist"
                  step="0.1"
                  min="0.1"
                  class="form-control"
                  placeholder="e.g. 1"
                />
                <select v-model="activeUnit" class="form-select">
                  <option value="km">km</option>
                  <option value="m">m</option>
                </select>
              </div>
            </div>

            <!-- Default Active Speed -->
            <div class="col-md-6">
              <label class="form-label fw-bold">Default Active Speed (km/h)</label>
              <input
                type="number"
                v-model="defaultActiveSpeed"
                step="0.1"
                min="0.1"
                class="form-control"
              />
            </div>

            <!-- Number of Intervals -->
            <div class="col-md-6">
              <label class="form-label fw-bold">Number of Active Intervals</label>
              <input
                type="number"
                v-model.number="numIntervals"
                min="1"
                step="1"
                class="form-control"
              />
            </div>

            <!-- Per-interval speeds -->
            <div v-for="(speed, index) in activeSpeeds" :key="index" class="col-12">
              <label class="form-label fw-bold">
                Active Speed for Interval {{ index + 1 }} (km/h)
              </label>
              <input
                type="number"
                v-model.number="activeSpeeds[index]"
                step="0.1"
                min="0.1"
                class="form-control"
              />
            </div>

            <!-- Rest Distance -->
            <div class="col-md-6">
              <label class="form-label fw-bold">Rest Distance</label>
              <div class="input-group">
                <input
                  type="number"
                  v-model="restDist"
                  step="0.1"
                  min="0.1"
                  class="form-control"
                />
                <select v-model="restUnit" class="form-select">
                  <option value="km">km</option>
                  <option value="m">m</option>
                </select>
              </div>
            </div>

            <!-- Rest Speed -->
            <div class="col-md-6">
              <label class="form-label fw-bold">Rest Speed (km/h)</label>
              <input
                type="number"
                v-model="restSpeed"
                step="0.1"
                min="0.1"
                class="form-control"
              />
            </div>
          </div>

          <div class="text-center mt-5">
            <button @click="start" class="btn btn-success btn-lg px-5">
              Start Workout
            </button>
          </div>
        </div>

        <!-- Timer View -->
        <div v-else class="card text-center p-5 shadow">
          <h2 class="mb-4" :class="currentPhase === 'active' ? 'text-success' : 'text-danger'">
            {{ currentPhase === 'active' ? 'ACTIVE' : 'REST' }}
          </h2>

          <div class="mb-4">
            <div class="fs-6 text-muted">Target Speed</div>
            <div class="display-4 fw-bold">{{ currentSpeed }} <small>km/h</small></div>
          </div>

          <div class="mb-4">
            <div class="fs-6 text-muted">Time Remaining</div>
            <div class="display-3 fw-bold text-primary">{{ formatTime(remainingSeconds) }}</div>
          </div>

          <div class="mb-4">
            <div class="fs-6 text-muted">Interval</div>
            <div class="fs-4 fw-bold">{{ currentInterval }} / {{ numIntervals }}</div>
          </div>

          <button @click="stopTimer" class="btn btn-outline-danger btn-lg px-5 mt-3">
            Stop Workout
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch } from 'vue';

export default defineComponent({
  name: 'IntervalRunningTimer',

  setup() {
    // Form inputs
    const activeDist = ref<number>(1);
    const activeUnit = ref<'km' | 'm'>('km');
    const defaultActiveSpeed = ref<number>(12.7);
    const numIntervals = ref<number>(4);
    const activeSpeeds = ref<number[]>(Array.from({ length: 4 }, () => 12.7));
    const restDist = ref<number>(500);
    const restUnit = ref<'km' | 'm'>('m');
    const restSpeed = ref<number>(6.0);

    // Timer state
    const isRunning = ref<boolean>(false);
    const currentPhase = ref<'active' | 'rest'>('active');
    const currentSpeed = ref<number>(0);
    const remainingSeconds = ref<number>(0);
    const currentInterval = ref<number>(1);
    const timerId = ref<NodeJS.Timeout | null>(null);
    const targetEndTime = ref<number>(0);

    // Computed distances in km
    const activeDistKm = computed<number>(() =>
      activeUnit.value === 'km' ? activeDist.value : activeDist.value / 1000
    );

    const restDistKm = computed<number>(() =>
      restUnit.value === 'km' ? restDist.value : restDist.value / 1000
    );

    const restTimeSeconds = computed<number>(() =>
      Math.round((restDistKm.value / restSpeed.value) * 3600)
    );

    // Update per-interval speeds when number of intervals changes
    watch(numIntervals, (newVal: number, oldVal: number) => {
      if (newVal > oldVal) {
        const toAdd = newVal - oldVal;
        for (let i = 0; i < toAdd; i++) {
          activeSpeeds.value.push(defaultActiveSpeed.value);
        }
      } else if (newVal < oldVal) {
        activeSpeeds.value.splice(newVal);
      }
    });

    const calculateActiveTime = (speed: number): number =>
      Math.round((activeDistKm.value / speed) * 3600);

    const start = () => {
      if (activeSpeeds.value.length === 0) return;

      currentInterval.value = 1;
      currentPhase.value = 'active';
      currentSpeed.value = activeSpeeds.value[0];
      remainingSeconds.value = calculateActiveTime(currentSpeed.value);
      targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
      isRunning.value = true;
      startTimer();
    };

    const stopTimer = () => {
      if (timerId.value) {
        clearInterval(timerId.value);
        timerId.value = null;
      }
      isRunning.value = false;
    };

    const startTimer = () => {
      timerId.value = setInterval(() => {
        const now = performance.now();
        const timeLeftMs = targetEndTime.value - now;
        const timeLeftSec = Math.max(0, Math.ceil(timeLeftMs / 1000));

        remainingSeconds.value = timeLeftSec;

        if (timeLeftSec <= 0) {
          if (timerId.value) clearInterval(timerId.value);
          switchPhase();
        }
      }, 100);
    };

    const switchPhase = () => {
      if (currentPhase.value === 'active') {
        // Switch to rest
        currentPhase.value = 'rest';
        currentSpeed.value = restSpeed.value;
        remainingSeconds.value = restTimeSeconds.value;
        targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
        startTimer();
      } else {
        // Next active interval or finish
        currentInterval.value++;
        if (currentInterval.value > numIntervals.value) {
          stopTimer();
        } else {
          currentPhase.value = 'active';
          currentSpeed.value = activeSpeeds.value[currentInterval.value - 1];
          remainingSeconds.value = calculateActiveTime(currentSpeed.value);
          targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
          startTimer();
        }
      }
    };

    const formatTime = (seconds: number): string => {
      const minutes = Math.floor(seconds / 60);
      const secs = seconds % 60;
      return `${minutes}:${secs.toString().padStart(2, '0')}`;
    };

    return {
      activeDist,
      activeUnit,
      defaultActiveSpeed,
      numIntervals,
      activeSpeeds,
      restDist,
      restUnit,
      restSpeed,
      isRunning,
      currentPhase,
      currentSpeed,
      remainingSeconds,
      currentInterval,
      start,
      stopTimer,
      formatTime,
    };
  },
});
</script>

<style scoped>
.card {
  border-radius: 1rem;
}

.display-3 {
  font-weight: 700;
  letter-spacing: -1px;
}
</style>
