<template>
  <div class="container">
    <div class="m-5">
      <div class="d-flex justify-content-between">
        <h2>Tarefas</h2>
        <button class="btn btn-primary" @click="openAddModal">Adicionar Tarefa</button>        
      </div>

      <div class="mx-auto text-center" v-if="loading">
        <div class="spinner-border text-primary" role="status">
          <span class="sr-only"></span>
        </div>
      </div>

      <ul class="list-group mt-3">
        <li v-for="task in tasks" :key="task.id" class="list-group-item">
          {{ task.title }} - {{ task.status }}
          <button class="btn btn-link" @click="openDetailsModal(task)">Ver Detalhes</button>
        </li>
      </ul>
    </div>

    <div v-if="selectedTask" class="modal-base">
      <div class="modal-base-content">
        <span class="close" @click="closeDetailsModal">&times;</span>
        <h3>Detalhes da Tarefa</h3>
        <p><strong>Título:</strong> {{ selectedTask.title }}</p>
        <p><strong>Descrição:</strong> {{ selectedTask.description }}</p>
        <p><strong>Data de Vencimento:</strong> {{ selectedTask.due_date }}</p>
        <p><strong>Status:</strong> {{ selectedTask.status }}</p>

        <div class="d-flex justify-content-between">
          <button class="btn btn-primary" @click="editTask(selectedTask)">Editar</button>
          <button class="btn btn-danger" @click="deleteTask(selectedTask)">Excluir</button>
        </div>
      </div>
    </div>

    <div v-if="editingTask" class="modal-base">
      <div class="modal-base-content">
        <span class="close" @click="cancelEdit">&times;</span>
        <h3>Editar Tarefa</h3>
        <form>
          <div class="form-group">
            <label for="edit-title">Título:</label>
            <input type="text" class="form-control" id="edit-title" v-model="objectTask.title" required>
          </div>

          <div class="form-group">
            <label for="edit-description">Descrição:</label>
            <textarea id="edit-description" class="form-control" v-model="objectTask.description" required></textarea>
          </div>

          <div class="form-group">
            <label for="edit-due-date">Data de Vencimento:</label>
            <input type="date" class="form-control" id="edit-due-date" v-model="objectTask.due_date" required>
          </div>

          <div class="form-group">
            <label for="edit-status">Status:</label>
            <select id="edit-status" class="form-control" v-model="objectTask.status" required>
              <option value="pending">Pendente</option>
              <option value="completed">Concluída</option>
              <option value="canceled">Cancelada</option>
            </select>
          </div>

          <div class="form-group mt-2 d-flex flex-row-reverse">
            <button class="btn btn-primary" @click="updateTask()">Salvar Alterações</button>
          </div>
        </form>
      </div>
    </div>

    <div v-if="addingTask" class="modal-base">
      <div class="modal-base-content">
        <span class="close" @click="cancelEdit">&times;</span>
        <h3>{{ addingTask ? 'Adicionar Tarefa' : 'Editar Tarefa' }}</h3>
        <form>
          <div class="form-group">
            <label for="edit-title">Título:</label>
            <input type="text" class="form-control" id="edit-title" v-model="objectTask.title" required>
          </div>

          <div class="form-group">
            <label for="edit-description">Descrição:</label>
            <textarea class="form-control" id="edit-description" v-model="objectTask.description" required></textarea>
          </div>

          <div class="form-group">
            <label for="edit-due-date">Data de Vencimento:</label>
            <input type="date" class="form-control" id="edit-due-date" v-model="objectTask.due_date" required>
          </div>

          <div class="form-group">
            <label for="edit-status">Status:</label>
            <select class="form-control" id="edit-status" v-model="objectTask.status" required>
              <option value="pending">Pendente</option>
              <option value="completed">Concluída</option>
              <option value="canceled">Cancelada</option>
            </select>
          </div>

          <div class="form-group mt-2 d-flex flex-row-reverse">
            <button class="btn btn-primary" @click="addNewTask()">Nova Tarefa</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tasks: [],
      selectedTask: null,
      editingTask: null,
      addingTask: null,
      loading: false,
      objectTask: {
        id: null,
        title: '',
        description: '',
        due_date: '',
        status: '',
      },
    };
  },
  mounted() {
    this.fetchTasks();
  },
  methods: {
    async fetchTasks() {
      this.loading = true;
      await axios.get('http://127.0.0.1:8000/api/tasks')
        .then(response => {
          this.tasks = response.data;
        })
        .catch(error => {
          console.error('Error fetching tasks', error);
        });

      this.loading = false;
    },
    openDetailsModal(task) {
      this.selectedTask = task;
    },
    closeDetailsModal() {
      this.selectedTask = null;
    },
    async deleteTask(task) {
      this.loading = true;

      await axios.delete(`http://127.0.0.1:8000/api/tasks/${task.id}`)
        .then(() => {
          this.fetchTasks();
          this.closeDetailsModal();
        })
        .catch(error => {
          console.error('Error deleting task', error);
        });

      this.loading = false;
    },
    editTask(task) {
      this.editingTask = task;
      this.objectTask.id = task.id;
      this.objectTask.title = task.title;
      this.objectTask.description = task.description;
      this.objectTask.due_date = task.due_date;
      this.objectTask.status = task.status;

      this.closeDetailsModal();
    },
    cancelEdit() {
      this.editingTask = null;
      this.addingTask = null;
      this.objectTask = {
        id: null,
        title: '',
        description: '',
        due_date: '',
        status: '',
      };
    },
    updateTask() {
      axios.put(`http://127.0.0.1:8000/api/tasks/${this.objectTask.id}`, this.objectTask)
        .then(() => {
          this.fetchTasks();
          this.editingTask = null;
          this.objectTask = {
            id: null,
            title: '',
            description: '',
            due_date: '',
            status: '',
          };
        })
        .catch(error => {
          console.error('Error updating task', error);
        });
    },
    openAddModal() {
      this.addingTask = true;
      this.objectTask = {
        id: null,
        title: '',
        description: '',
        due_date: '',
        status: 'pending',
      };
    },
    addNewTask() {
      axios.post('http://127.0.0.1:8000/api/tasks', this.objectTask)
        .then(() => {
          this.fetchTasks();
          this.cancelEdit();
        })
        .catch(error => {
          console.error('Error adding new task', error);
        });
    },
  }
};
</script>

<style scoped>
.modal-base {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0,0,0);
  background-color: rgba(0,0,0,0.4);
}

.modal-base-content {
  background-color: #fefefe;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
}

.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}

</style>

