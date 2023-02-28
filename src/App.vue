<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container class="q-pa-md">
      <h1 class="text-center q-my-xs primary">{{ timer }}</h1>
      <q-dialog v-model="showInputDialog">
        <q-card>
          <q-card-section>
            <q-input
              v-model="myNumberInput"
              type="number"
              label="Wprowadź swoją cyfrę"
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
          :color="value == answer[index] ? 'green' : 'red'"
          outlined
          :style="inputStyle(value, index)"
          maxlength="1"
          class="col-1 q-ma-md p-ma-xs"
        />
      </div>
      <div v-if="myNumber && answer.join('') == myNumber">
        <div>Dobrze!</div>
        <q-btn color="primary" class="q-mx-auto full-width" @click="reset"
          >Reset</q-btn
        >
      </div>
      <q-btn color="primary" class="full-width" @click="showInputDialog = true">
        Start
      </q-btn>
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";

const myNumber = ref<string>("");
const answer = ref<string[]>([]);
const timer = ref<number>(60);
const intervalId = ref<number>(0);
const showInputDialog = ref<boolean>(false);
const timeInput = ref<number>(60);

const splitedNumber = computed(() => myNumber.value.split(""));
const myNumberInput = computed({
  set(val: string) {
    if (val.match(/\d+/)) {
      myNumber.value = val;
      return;
    }
  },
  get() {
    return myNumber.value;
  },
});

const inputStyle = (value: string, index: number) => ({
  backgroundColor: value == answer.value[index] ? "lightgreen" : "red",
  fontSize: "2rem",
  width: "3rem",
});

const startGame = () => {
  if (intervalId.value > 0) {
    return;
  }
  timer.value = timeInput.value;
  intervalId.value = setInterval(() => {
    --timer.value;
  }, 1000);
};

const reset = () => {
  myNumber.value = "";
  answer.value = [];
  timer.value = 60;
  clearInterval(intervalId.value);
  intervalId.value = 0;
  startGame();
};

watch(timer, (val) => {
  if (val <= 0) {
    clearInterval(intervalId.value);
  }
});

watch(
  answer,
  (val) => {
    if (val.join("") == myNumber.value) {
      clearInterval(intervalId.value);
    }
  },
  { deep: true }
);
</script>
<style>
.q-dialog__inner {
  min-width: 10000px;
}

</style>
