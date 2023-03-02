<template>
  <q-layout
    view="lHh Lpr lFf"
    @keyup.ctrl.enter="showResetButton = !showResetButton"
  >
    <q-page-container class="q-pa-md">
      <h1 class="text-center q-my-xs primary">
        {{ displayTime }}
      </h1>
      <q-dialog v-model="showInputDialog">
        <q-card style="width: 700px; max-width: 80vw">
          <q-card-section>
            <q-input
              v-model="myNumberInput"
              type="number"
              label="Wprowadź swoją cyfrę"
              :rules="[(val) => !!val || 'Field is required']"
            />
            <q-input v-model="timeInput" type="number" label="Czas (s)" />
          </q-card-section>
          <q-card-actions align="right">
            <q-btn label="Cancel" color="primary" v-close-popup />
            <q-btn
              label="Ok"
              color="primary"
              @click="startGame"
              v-close-popup
            />
          </q-card-actions>
        </q-card>
      </q-dialog>
      <div class="row justify-around">
        <q-input
          v-for="(value, index) in splitedNumber"
          :key="value + index"
          v-model="answer[index]"
          outlined
          :style="inputStyle(value, index)"
          maxlength="1"
          class="col-1 q-ma-md p-ma-xs"
          :readonly="!runed"
        />
      </div>
      <h2
        v-if="myNumber && answer.join('') == myNumber"
        style="text-align: center; color: green"
      >
        GRATULACJE!
      </h2>
      <h2 v-if="timeOver" style="text-align: center; color: red">KONIEC GRY</h2>

      <q-btn
        v-if="showResetButton"
        color="primary"
        class="q-mx-auto full-width"
        @click="reset"
      >
        Reset
      </q-btn>
      <q-btn
        v-if="!hideStart"
        color="primary"
        class="full-width"
        @click="showInputDialog = true"
      >
        Start
      </q-btn>
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";

const milisecondsToSeconds = (miliseconds: number) =>
  Math.floor(miliseconds / 1000);

const myNumber = ref<string>("");
const answer = ref<string[]>([]);
const timer = ref<number>(60);
const intervalId = ref<number>(0);
const showInputDialog = ref<boolean>(false);
const showResetButton = ref<boolean>(false);
const timeInput = ref<number>(60);
const runed = ref<boolean>(false);
const timeOver = ref<boolean>(false);
const hideStart = ref<boolean>(false);

const splitedNumber = computed(() => myNumber.value.split(""));
const myNumberInput = computed({
  set(val: string) {
    if (val.match(/\d+/)) {
      myNumber.value = val;
      return;
    }
  },
  get() {
    return myNumber.value || "0";
  },
});

const twoSigns = (nm: number) => (nm > 9 ? nm + "" : "0" + nm);

const displayTime = computed(() => {
  const minutes = Math.floor(timer.value / 60);
  const secunds = timer.value % 60;

  return twoSigns(minutes) + ":" + twoSigns(secunds);
});

const inputStyle = (value: string, index: number) => ({
  // backgroundColor: value == answer.value[index] ? "lightgreen" : "red",
  fontSize: "2rem",
  width: "3rem",
});

const startInterval = (startTime: number) => {
  runed.value = true;
  hideStart.value = true;
  timer.value = milisecondsToSeconds(
    startTime - Date.now() + timeInput.value * 1000
  );

  intervalId.value = setInterval(() => {
    --timer.value;
  }, 1000);
};

const startGame = () => {
  if (intervalId.value > 0) {
    return;
  }

  const startTime = Date.now();
  timer.value = timeInput.value;

  localStorage.setItem("myNumber", myNumber.value);
  localStorage.setItem("timeInput", timeInput.value + "");
  localStorage.setItem("startTime", startTime + "");

  startInterval(startTime);
};

const reset = () => {
  myNumber.value = "";
  answer.value = [];
  runed.value = false;
  hideStart.value = false;
  timeOver.value = false;
  clearInterval(intervalId.value);
  showResetButton.value = false;
  intervalId.value = 0;
};

watch(timer, (val) => {
  if (val <= 0) {
    clearInterval(intervalId.value);
    // localStorage.clear();
    runed.value = false;
    timeOver.value = true;
  }
});

watch(
  answer,
  (val) => {
    if (val.join("") == myNumber.value) {
      clearInterval(intervalId.value);
      runed.value = false;
    }
  },
  { deep: true }
);

onMounted(() => {
  const storedStartTime = localStorage.getItem("startTime");
  const storagedMyNymber = localStorage.getItem("myNumber");
  const storagedTimeInput = localStorage.getItem("timeInput");

  if (storedStartTime && storagedMyNymber && storagedTimeInput) {
    myNumber.value = storagedMyNymber;
    timeInput.value = +storagedTimeInput;

    startInterval(+storedStartTime);
  }
});
</script>
