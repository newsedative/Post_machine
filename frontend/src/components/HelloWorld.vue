<script setup>
import {onMounted, ref} from "vue"

const commandsData = ref({})
let showForm = ref(false)
let commandNumber = ref(1)
let command = ref({
  number: '',
  command_type: '',
  command_value: '',
  transition: '',
  comment: ''
})

const commandsType = [
  {
    value: 'right', name: '-> (Сдвиг вправо)'
  },
  {
    value: 'left', name: '<- (Сдвиг влево)'
  },
  {
    value: 'label', name: 'V (поставить метку)'
  },
  {
    value: 'delete', name: 'X (удалить метку)'
  },
  {
    value: 'condition', name: '? (переход)'
  },
  {
    value: 'end', name: '! (конец)'
  }
]

const inputStyle = `
width: 50px;
height: 50px;
background-color: #2c3e50;
color: white;
border-radius: 6px;
padding-right: 5px;
padding-left: 20px;
font-size: 20px;`

const inputStyleCarriage = `
width: 50px;
height: 50px;
background-color: red;
color: white;
border-radius: 6px;
padding-right: 5px;
padding-left: 20px;
font-size: 20px;`

onMounted(() => {
  for (let i = 0; i < 21; i++) {
    let mainBody = document.querySelector('.main-tape')
    let inputField = document.createElement('input')
    inputField.setAttribute('type', 'text')
    inputField.setAttribute('maxlength', '1')
    inputField.setAttribute('data-index', i);
    inputField.className = 'main-tape__input'
    inputField.style.cssText = inputStyle
    mainBody.append(inputField)
  }
  let carriage = document.querySelector(`.main-tape__input[data-index="${9}"]`)
  carriage.classList.add('carriage')
  carriage.style.cssText = inputStyleCarriage
})

function createCommand() {
  command.value.number = commandNumber.value
  let newCommand = command.value
  for (let elem of commandsType) {
    if (elem.value === command.value.command_type) {
      newCommand.command_type = elem.name
      newCommand.command_value = elem.value
    }
  }
  commandsData.value[command.value.number] = newCommand
  commandNumber.value += 1
  command.value = {
    number: '',
    command_type: '',
    transition: '',
    comment: ''
  }
}

function changeCarriage(indexBlock, pastBlock) {
  pastBlock.classList.remove('carriage')
  pastBlock.style.cssText = inputStyle
  let carriageNext = document.querySelector(`.main-tape__input[data-index="${indexBlock}"]`)
  carriageNext.classList.add('carriage')
  carriageNext.style.cssText = inputStyleCarriage
}
function rightStep(curStep, curBlock, indexBlock) {
  changeCarriage(Number(indexBlock) + 1, curBlock)
  return curStep.transition
}

function leftStep(curStep, curBlock, indexBlock) {
  changeCarriage(Number(indexBlock) - 1, curBlock)
  return curStep.transition
}

function labelStep(curStep, curBlock) {
  curBlock.value = 'v'
  return curStep.transition
}
function deleteStep(curStep, curBlock) {
  curBlock.value = ''
  return curStep.transition
}
function conditionStep(curStep, curBlock) {
  const numbers = curStep.transition.split('; ');
  const firstNumber = parseInt(numbers[0]);
  const secondNumber = parseInt(numbers[1]);
  if (curBlock.value === 'v') {
    return secondNumber
  } else {
    return firstNumber
  }
}
function endStep() {
  return 1
}

function startMachine() {
  let flag = true
  let currentBlock = ''
  let blockIndex = 0
  let currentStep = 1
  let currentCommand = ''
  let commandType = ''
  while (flag) {
    currentBlock = document.querySelector(`.carriage`)
    currentCommand = commandsData.value[currentStep]
    blockIndex = currentBlock.dataset.index
    commandType = currentCommand.command_value
    if (commandType === 'right') {
      currentStep = rightStep(currentCommand, currentBlock, blockIndex)
    } else if (commandType === 'left') {
      currentStep = leftStep(currentCommand, currentBlock, blockIndex)
    } else if (commandType === 'label') {
      currentStep = labelStep(currentCommand, currentBlock)
    } else if (commandType === 'delete') {
      currentStep = deleteStep(currentCommand, currentBlock)
    } else if (commandType === 'condition') {
      currentStep = conditionStep(currentCommand, currentBlock)
    } else if (commandType === 'end') {
      flag = false
    } else {
      flag = false
    }
    currentStep = Number(currentStep)
  }
}

</script>

<template>
  <div class="main">
    <div class="main-title">
      <h1>Реализация Машины Поста </h1>
    </div>
    <div class="main-tape">
      <!--<input type="text" maxlength="1" class="main-tape__input">-->
    </div>
    <div class="main-commands">
      <v-dialog max-width="500">
        <template v-slot:activator="{ props: activatorProps }">
          <div class="main-commands__btns">
            <v-btn variant="outlined"
                   class="main-commands__btn"
                   v-bind="activatorProps"
                   text="Добавить команду"
                   @click="showForm=true"
            ></v-btn>
            <v-btn variant="outlined"
              class="main-commands__btn"
              text="Старт"
              @click="startMachine"
            ></v-btn>
          </div>
        </template>

        <template v-slot:default="{ isActive }" class="main-commands__dialog">
          <v-card title="Добавление команды">
            <v-card-actions>
              <v-select
                  v-model="command.command_type"
                  label="Команда"
                  :items="commandsType"
                  item-value="value"
                  item-title="name"
              ></v-select>
              <v-text-field
                v-model="command.transition"
                :counter="10"
                label="Переход"
                required
              ></v-text-field>
              <v-text-field
                v-model="command.comment"
                :counter="30"
                label="Комментарий"
                required
              ></v-text-field>
              <div class="main-commands__dialog__btns">
                <v-btn variant="outlined"
                   text="Добавить"
                   @click="createCommand"
                   type="submit"
                ></v-btn>
                <v-btn variant="outlined"
                   text="Закрыть"
                   @click="isActive.value = false"
                ></v-btn>
              </div>
            </v-card-actions>
          </v-card>
        </template>
      </v-dialog>

      <v-table height="300px" class="main-commands__table">
        <thead class="main-commands__table__headers">
          <tr>
            <th class="text-left">Номер</th>
            <th class="text-left">Команда</th>
            <th class="text-left">Переход</th>
            <th class="text-left">Комментарий</th>
          </tr>
        </thead>
        <tbody class="main-commands__table__body">
          <tr
              v-for="item in commandsData"
              class="main-commands__table__body__tr"
          >
            <td>{{ item.number }}</td>
            <td>{{ item.command_type }}</td>
            <td>{{ item.transition }}</td>
            <td>{{ item.comment }}</td>
          </tr>
        </tbody>
      </v-table>
    </div>
  </div>
</template>

<style scoped>
/*.main-tape__input {
  width: 50px;
  height: 50px;
  background-color: #2c3e50;
  color: white;
  border-radius: 6px;
  padding-right: 5px;
  padding-left: 5px;
  font-size: 20px;
}*/

.main {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.main-tape {
  display: flex;
  flex-direction: row;
  gap: 8px;
}
.v-card-actions {
  display: inline;
}
.main-commands__btns {
  display: flex;
  gap: 16px;
}
.main-commands__dialog__btns {
  display: flex;
  gap: 16px;
}
</style>
