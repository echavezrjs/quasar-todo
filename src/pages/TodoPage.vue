<template>
  <q-page class="bg-grey-3 column">
    <div class="row q-pa-sm bg-primary">
      <q-input 
      class="col"
        v-model="newTask" 
        @keyup.enter="addTask"
        square
        filled 
        bg-color="white"
        placeholder="Add task" 
        dense>
        <template v-slot:append>
          <q-btn 
            @click="addTask"
            round 
            dense 
            flat 
            icon="add" />
        </template>
      </q-input>
    </div>
    <q-list 
      class="bg-white"
      separator
      bordered>
      <q-item
        v-for="task in orderedTask"
        :key="task.id"
        :class="{ 'done bg-blue-1' : task.done }"
        v-ripple>
        <q-item-section avatar>
          <q-checkbox 
            v-if="!task.edit"
            @click="addDoneDate(task.id)"
            v-model="task.done" 
            color="primary" />
        </q-item-section>
        <q-item-section>
          <q-item-label>
            <q-input 
              v-if="task.edit"
              class="col"
              autofocus
              v-model="task.changeTitle" 
              @keyup.enter="updateTask(task.id)"
              square
              filled 
              bg-color="white" 
              dense>
              <template v-slot:append>
                <q-btn 
                  @click="updateTask(task.id)"
                  round 
                  dense 
                  flat 
                  icon="edit" />
                  <q-btn 
                  @click="cancelEdit(task.id)"
                  round 
                  dense 
                  flat 
                  icon="cancel" />
              </template>
            </q-input>
            <template v-if="!task.edit">{{ task.title }}</template>
          </q-item-label>
        </q-item-section>
        <q-item-section
          v-if="!task.done && !task.edit"
          side>
          <q-btn 
            @click.stop="editTask(task.id)"
            flat 
            round 
            dense 
            color="primary" 
            icon="edit" />
        </q-item-section>
        <q-item-section
          v-if="!task.done && !task.edit"
          side>
          <q-btn 
            @click.stop="deleteTask(task.id)"
            flat 
            round 
            dense 
            color="primary" 
            icon="delete" />
        </q-item-section>
      </q-item>
    </q-list>
    <div
      v-if="!tasks.length" 
      class="no-tasks absolute-center">
      <q-icon 
        name="check"
        size="100px"
        color="primary" />
      <div class="text-h5 text-primary text-center">
        No tasks
      </div>
    </div>
  </q-page>
</template>

<script>
import { v4 as uuidv4 } from 'uuid';

export default ({
  name: 'HelpPage',
  data() {
    return {
      newTask: '',
      tasks: []
    }
  },
  computed: {
    orderedTask() {
      return this.tasks.slice().sort((a, b) => { return a.dateDone - b.dateDone})
    }
  },
  methods: {
    deleteTask(id) {
      this.$q.dialog({
        title: 'Confirm',
        message: 'Would you like to delete the task?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        this.tasks = this.tasks.filter(data => data.id != id)
        this.$q.notify('Task deleted.')
      })
    },
    addTask() {
      if(this.newTask)
      {
        if(this.tasks.some(data => data.title.toLowerCase() === this.newTask.toLowerCase())){
          this.$q.dialog({
            title: 'Alert',
            message: 'Task already exist'
          })
          return
        }
        this.tasks.push({
          id: uuidv4(),
          title: this.newTask,
          changeTitle: '',
          done: false,
          dateDone: null,
          edit: false
        })
        this.newTask = ''
      }
    },
    editTask(id) {
      const data = this.tasks.find(task => task.id === id)
      data.changeTitle = data.title
      data.edit = !data.edit
    },
    cancelEdit(id) {
      const data = this.tasks.find(task => task.id === id)
      data.edit = !data.edit
    },
    updateTask(id) {
      const data = this.tasks.find(task => task.id === id)
      if(data.changeTitle){
        if(this.tasks.some(task => task.title.toLowerCase() ===  data.changeTitle.toLowerCase()) && data.changeTitle != data.title){
          this.$q.dialog({
            title: 'Alert',
            message: 'Duplicate task.'
          })
          return
        }
        data.title = data.changeTitle
        data.edit = !data.edit
      }
    },
    addDoneDate(id) {
      this.tasks = this.tasks.map(data => {
        if(data.id === id) data.dateDone = data.dateDone ? null : Date.now()
        return data
      })
    }
  }
})
</script>

<style lang="scss">
.done {
  .q-item__label{
    text-decoration: line-through;
    color: #bbb;
  }
}
.no-tasks{
  opacity: 0.5;
}
</style>