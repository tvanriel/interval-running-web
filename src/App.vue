<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-lg-8">
        <h1 class="text-center mb-5">Interval Running Timer</h1>

        <!-- Setup Form -->
        <div v-if="!isRunning" class="card p-4 shadow">
          <div class="row g-4">

            <div class="card border-warning">
              <div class="card-body">
                <div class="row">

                  <!-- Warmup -->
                  <div class="col-12">
                    <div class="form-check form-switch mb-3">
                      <input class="form-check-input" type="checkbox" id="warmupEnabled" v-model="warmupEnabled" />
                      <label class="form-check-label fw-bold" for="warmupEnabled">Include <span class="text-warning">Warmup</span></label>
                    </div>
                  </div>


                  <div v-if="warmupEnabled" class="col-md-6">
                    <label class="form-label fw-bold"><span class="text-warning">Warmup</span> Distance</label>
                    <div class="input-group">
                      <input
                          type="number"
                          v-model="warmupDist"
                          step="0.1"
                          min="0.1"
                          class="form-control"
                          />
                      <select v-model="warmupUnit" class="form-select">
                        <option value="km">km</option>
                        <option value="m">m</option>
                      </select>
                    </div>
                  </div>
                </div>

                <div v-if="warmupEnabled" class="col-md-6">
                  <label class="form-label fw-bold"><span class="text-warning">Warmup</span> Speed</label>
                  <input
                      type="number"
                      v-model="warmupSpeed"
                      step="0.1"
                      min="0.1"
                      class="form-control"
                      />
                </div>



              </div>
            </div>

            <div class="card border-success">
              <div class="card-body">
                <div class="row">

                  <!-- Active Distance -->
                  <div class="col-md-6">
                    <label class="form-label fw-bold"><span class="text-success">Active</span> Distance</label>
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
                    <label class="form-label fw-bold">Default <span class="text-success">Active</span> Speed</label>
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
                    <label class="form-label fw-bold">Number of <span class="text-success">Active</span> Intervals</label>
                    <input
                        type="number"
                        v-model.number="numIntervals"
                        min="1"
                        step="1"
                        class="form-control"
                        />
                  </div>

                  <!-- Per-interval speeds -->
                  <div v-for="(, index) in activeSpeeds" :key="index" class="col-12">
                    <label class="form-label fw-bold">
                      <span class="text-success">Active</span> Speed for Interval {{ index + 1 }}
                    </label>
                    <input
                        type="number"
                        v-model.number="activeSpeeds[index]"
                        step="0.1"
                        min="0.1"
                        class="form-control"
                        />
                  </div>



                </div>
              </div>
            </div>

            <div class="card border-danger">
              <div class="card-body">
                <div class="row">
                  <!-- Rest Distance -->
                  <div class="col-md-6">
                    <label class="form-label fw-bold"><span class="text-danger">Rest</span> Distance</label>
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
                    <label class="form-label fw-bold"><span class="text-danger">Rest</span> Speed (km/h)</label>
                    <input
                        type="number"
                        v-model="restSpeed"
                        step="0.1"
                        min="0.1"
                        class="form-control"
                        />
                  </div>
                </div>

              </div>


            </div>
          </div>
          <div class="text-center mt-5">
            <button @click="start" class="btn btn-success btn-lg px-5">
              Start Workout
            </button>
          </div>
        </div>

        <!-- Running View -->
        <div v-else class="card text-center p-5 shadow">
          <h2
              class="mb-4"
              :class="{
                      'text-success': currentPhase === 'active',
                      'text-warning': currentPhase === 'warmup',
                      'text-danger': currentPhase === 'rest'
                      }"
              >
              {{ phaseName }}
          </h2>

            <div class="mb-4">
              <div class="fs-6 text-muted">Time Remaining</div>
              <div class="display-1 fw-bold text-info font-monospace">{{ formatTime(remainingSeconds) }}</div>
            </div>

            <div class="mb-4">
              <div class="fs-6 text-muted">Target Speed</div>
              <div class="display-4 fw-bold"><span class="font-monospace">{{ currentSpeed }}</span> <small>km/h</small></div>
            </div>

            <div class="mb-4">
              <div class="fs-6 text-muted">Interval</div>
              <div class="fs-4 fw-bold">
                {{ intervalDisplay }}
              </div>
            </div>

            <div class="mt-4">
              <button
                  v-if="isPaused"
                  @click="resume"
                  class="btn btn-primary btn-lg px-5 me-3"
                  >
                  Resume
              </button>
                <button v-else @click="pause" class="btn btn-warning btn-lg px-5 me-3">
                  Pause
                </button>
                <button @click="stopTimer" class="btn btn-outline-danger btn-lg px-5">
                  Stop
                </button>
            </div>
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
    const warmupEnabled = ref<boolean>(true);
    const warmupDist = ref<number>(1);
    const warmupUnit = ref<'km' | 'm'>('km');
    const warmupSpeed = ref<number>(8.0);

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
    const isPaused = ref<boolean>(false);
    const currentPhase = ref<'warmup' | 'active' | 'rest'>('warmup');
    const currentSpeed = ref<number>(0);
    const remainingSeconds = ref<number>(0);
    const currentInterval = ref<number>(0); // 0 = warmup, 1..n = active intervals
    const timerId = ref<number | null>(null);
    const targetEndTime = ref<number>(0);

    // Computed distances in km
    const warmupDistKm = computed<number>(() =>
      warmupUnit.value === 'km' ? warmupDist.value : warmupDist.value / 1000
    );

    const activeDistKm = computed<number>(() =>
      activeUnit.value === 'km' ? activeDist.value : activeDist.value / 1000
    );

    const restDistKm = computed<number>(() =>
      restUnit.value === 'km' ? restDist.value : restDist.value / 1000
    );

    const warmupTimeSeconds = computed<number>(() =>
      Math.round((warmupDistKm.value / warmupSpeed.value) * 3600)
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

    const calculateTime = (distKm: number, speed: number): number =>
      Math.round((distKm / speed) * 3600);

    const start = () => {
      if (activeSpeeds.value.length === 0) return;

      currentInterval.value = warmupEnabled.value ? 0 : 1;
      currentPhase.value = warmupEnabled.value ? 'warmup' : 'active';
      currentSpeed.value = (warmupEnabled.value ? warmupSpeed.value : activeSpeeds.value[0]) as number;

      remainingSeconds.value = warmupEnabled.value
        ? warmupTimeSeconds.value
        : calculateTime(activeDistKm.value, currentSpeed.value);

      targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
      isRunning.value = true;
      isPaused.value = false;
      startTimer();
    };

    const pause = () => {
      if (timerId.value) {
        clearInterval(timerId.value);
        timerId.value = null;
      }
      isPaused.value = true;
    };

    const resume = () => {
      targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
      isPaused.value = false;
      startTimer();
    };

    const stopTimer = () => {
      if (timerId.value) {
        clearInterval(timerId.value);
        timerId.value = null;
      }
      isRunning.value = false;
      isPaused.value = false;
      currentPhase.value = 'warmup';
      currentInterval.value = 0;
    };

    const startTimer = () => {
      timerId.value = setInterval(() => {
        const now = performance.now();
        const timeLeftMs = targetEndTime.value - now;
        const timeLeftSec = Math.max(0, Math.ceil(timeLeftMs / 1000));

        remainingSeconds.value = timeLeftSec;

        if (timeLeftSec <= 0) {
          if (timerId.value) clearInterval(timerId.value);
          timerId.value = null;
          switchPhase();
        }
      }, 100); // more frequent updates for smoother countdown
    };

    const switchPhase = () => {
      if (currentPhase.value === 'warmup') {
        // After warmup → first active
        currentInterval.value = 1;
        currentPhase.value = 'active';
        currentSpeed.value = activeSpeeds.value[0] as number;
        remainingSeconds.value = calculateTime(activeDistKm.value, currentSpeed.value);
      } else if (currentPhase.value === 'active') {
        // After active → rest
        currentPhase.value = 'rest';
        currentSpeed.value = restSpeed.value;
        remainingSeconds.value = restTimeSeconds.value;
      } else {
        // After rest → next active or finish
        currentInterval.value++;
        if (currentInterval.value > numIntervals.value) {
          stopTimer();
          return;
        }
        currentPhase.value = 'active';
        currentSpeed.value = activeSpeeds.value[currentInterval.value - 1] as number;
        remainingSeconds.value = calculateTime(activeDistKm.value, currentSpeed.value);
      }

      targetEndTime.value = performance.now() + remainingSeconds.value * 1000;
      startTimer();
    };

    const formatTime = (seconds: number): string => {
      const minutes = Math.floor(seconds / 60);
      const secs = seconds % 60;
      return `${minutes}:${secs.toString().padStart(2, '0')}`;
    };

    const phaseName = computed(() => {
      switch (currentPhase.value) {
        case 'warmup':
          return 'WARMUP';
        case 'active':
          return 'ACTIVE';
        case 'rest':
          return 'REST';
        default:
          return '';
      }
    });

    const intervalDisplay = computed(() => {
      if (currentPhase.value === 'warmup') return 'Warmup';
      if (currentPhase.value === 'rest') return `${currentInterval.value} / ${numIntervals.value}`;
      return `${currentInterval.value} / ${numIntervals.value}`;
    });

    return {
      warmupEnabled,
      warmupDist,
      warmupUnit,
      warmupSpeed,
      activeDist,
      activeUnit,
      defaultActiveSpeed,
      numIntervals,
      activeSpeeds,
      restDist,
      restUnit,
      restSpeed,
      isRunning,
      isPaused,
      currentPhase,
      currentSpeed,
      remainingSeconds,
      currentInterval,
      start,
      pause,
      resume,
      stopTimer,
      formatTime,
      phaseName,
      intervalDisplay,
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
