<script setup>
import json from '../src/assessment.json'
import { reactive, ref } from 'vue';

const index = ref(0);

const element = ref({});

const start = ref(false);

const end = ref(false);

const ans = ref("");

const evaluation = ref([
  {
    area: "Alfabetizzazione su informazione e dati",
    level: 0
  },
  {
    area: "Comunicazione e collaborazione",
    level: 0
  },
  {
    area: "Creazione di contenuti digitali",
    level: 0
  },
  {
    area: "Sicurezza",
    level: 0
  },
  {
    area: "Risolvere i problemi",
    level: 0
  }
])

function startQuiz(){
  element.value = json[index.value]
  start.value = true
  end.value = false
}

function nextQuestion(){
  element.value.ansId = ans.value;
  if((index.value + 1) >= json.length){
    endQuiz();
    return;
  }
  index.value++;
  ans.value = "";
  element.value = json[index.value];
}

function endQuiz(){
  end.value = true;
  start.value = false;
  evaluate()
}

function getAnsPoints(id, element){
  const foundAnswer = element.answers.find(ans => ans.id === id);
  if (foundAnswer) {
    return foundAnswer.points;
  }
  return 0;
}


function evaluate(){
  for(let i = 1; i <= 5; i++){
    console.log("COMPETENCE AREA " + i)
    let sum = 0.0;
    let n = 0.0;
    json.forEach(element => {
      if(element.area == i){
        sum += getAnsPoints(element.ansId, element);
        n++;
      }
    });

    if(n <= 0){
      evaluation.value[i-1].level = 1;
    }
    else{
      let parts = n / 8;
      for(let j = 8; j >= 1; j--){
        if(parts * j <= sum){
          evaluation.value[i-1].level = j;
          break;
        }
      }
    }
    
    console.log("total: " + n)
    console.log("sum: " + sum)
  }
}

function levelName(level) {
  switch(true) {
    case level < 3:
      return "Base";
    case level < 5:
      return "Intermedio";
    case level < 7:
      return "Avanzato";
    case level < 9:
      return "Altamente Specializzato";
    default:
      return "Unknown";
  }
}

</script>


<template> 

  <v-container style="background-color: lightgray;" class="rounded-b-xl">
    <v-row>
      <v-col cols="3"/>
      <v-col cols="6" class="text-h3 d-flex justify-center">
        DigComp Assessment Tool
      </v-col>
      <v-col cols="3"/>
    </v-row>
  </v-container>

  <v-container style="background-color: lightgray;" class="margin-custom rounded-xl" v-if="!start && !end">
    <v-row>
      <v-col cols="2"/>
      <v-col cols="8" class="d-flex justify-center">
        <div class="text-h4">Valuta le tue competenze digitali</div>
      </v-col>
      <v-col cols="2"/> 
    </v-row>
    
    <v-row>
      <v-col cols="4"/>
      <v-col cols="4" class="d-flex justify-center">
        <v-btn color="primary" @click="startQuiz()">INZIA</v-btn>
      </v-col>
      <v-col cols="4"/> 
    </v-row>
  </v-container>

  <v-container class="margin-custom" v-if="start">
    <v-row style="background-color: lightgray;" class="rounded-ts-xl rounded-te-xl">
      <v-col cols="3"/>
      <v-col cols="6">
        <div class="text-overline font-weight-bold">
          {{ evaluation.at(element.area - 1).area }}
        </div>
        <div class="text-h6">
          {{ element.question }}
        </div>
      </v-col>
      <v-col cols="3"/>
    </v-row>

    <v-row style="background-color: lightgray;" class="rounded-bs-xl rounded-be-xl">
      <v-col cols="3"/>
      <v-col cols="6" class="d-flex justify-center">
        <v-radio-group v-model="ans">
          <v-radio color="primary" v-for="choice in element.answers" :label="choice.choice" :value="choice.id"></v-radio>
        </v-radio-group>
      </v-col>
      <v-col cols="3"/>
    </v-row>

    <v-row>
      <v-col cols="4"/>
      <v-col cols="4" class="d-flex justify-center">
        <v-btn :disabled="!ans" @click="nextQuestion()" color="primary">AVANTI</v-btn>
      </v-col>
      <v-col cols="4"/>
    </v-row>
  </v-container>

  <v-container class="margin-custom-end" v-if="end && !start">
    <v-row>
      <v-col cols="3"/>
      <v-col cols="6" class="d-flex justify-center">
        <div class="text-h4">
          Le tue competenze digitali sono
        </div>
      </v-col>
      <v-col cols="3"/>
    </v-row>

    <v-row >
      <v-col cols="4"/>
      <v-col cols="4" class="d-flex justify-center">
        <v-expansion-panels class="my-4" variant="popout">
          <v-expansion-panel v-for="i in evaluation.length" :key="i">
            <v-expansion-panel-title expand-icon="mdi-menu-down">
              <p class="font-weight-bold">{{ evaluation[i-1].area + " -" }}</p>
              <p>{{ '&nbsp' + levelName(evaluation[i-1].level) + '&nbsp' + evaluation[i-1].level }}</p>
            </v-expansion-panel-title>
            <v-expansion-panel-text>
              <v-rating v-model="evaluation[i-1].level" readonly active-color="primary" length="8"></v-rating>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>
      <v-col cols="4"/>
    </v-row>
  </v-container>
</template>

<style scoped>
.margin-custom{
  margin-top: 8%;
}

.margin-custom-end{
  margin-top: 6%;
}
</style>