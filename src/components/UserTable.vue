<template>
  <div>
    <div class="ag-theme-alpine" style="height: 400px; width: 100%;" ref="gridContainer"></div>
    <form @submit.prevent="addUser">
      <input v-model="newUser.name" placeholder="Имя" required>
      <input v-model="newUser.gender" placeholder="Пол" required>
      <input v-model="newUser.email" placeholder="Почта" required>
      <input v-model="newUser.age" type="number" placeholder="Возраст" required>
      <button type="submit">Добавить</button>
    </form>
    <ul>
      <li v-for="log in logs" :key="log">{{ log }}</li>
    </ul>
  </div>
</template>

<script>
import { Grid } from 'ag-grid-community';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';

export default {
  data() {
    return {
      columnDefs: [
        { headerName: 'Имя', field: 'name.first' },
        { headerName: 'Пол', field: 'gender' },
        { headerName: 'Почта', field: 'email' },
        { headerName: 'Возраст', field: 'dob.age' },
        {
          headerName: 'Действие',
          cellRenderer: (params) => {
            let eDiv = document.createElement('div');
            eDiv.innerHTML = `<button style="color: red;">Удалить</button>`;
            eDiv.addEventListener('click', () => this.deleteUser(params));
            return eDiv;
          },
        },
      ],
      rowData: [],
      gridOptions: null,
      newUser: {
        name: '',
        gender: '',
        email: '',
        age: '',
      },
      logs: [],
    };
  },
  mounted() {
    this.gridOptions = {
      columnDefs: this.columnDefs,
      rowData: this.rowData,
    };

    this.grid = new Grid(this.$refs.gridContainer, this.gridOptions);

    // Fetch initial data
    fetch('https://randomuser.me/api/?results=5')
      .then(response => response.json())
      .then(data => {
        this.rowData = data.results;
        this.gridOptions.api.applyTransaction({ add: this.rowData }); // Добавление данных с помощью applyTransaction
      });
  },
  methods: {
    addUser() {
      const newUser = {
        name: { first: this.newUser.name },
        gender: this.newUser.gender,
        email: this.newUser.email,
        dob: { age: this.newUser.age },
      };
      this.rowData.push(newUser);
      this.gridOptions.api.applyTransaction({ add: [newUser] }); // Добавление нового пользователя с помощью applyTransaction
      this.logs.push(`Добавлен пользователь "${newUser.name.first}"`);
      this.newUser = {
        name: '',
        gender: '',
        email: '',
        age: '',
      };
    },
    deleteUser(params) {
      const index = this.rowData.indexOf(params.data);
      if (index > -1) {
        const removedUser = this.rowData[index];
        this.rowData.splice(index, 1);
        this.gridOptions.api.applyTransaction({ remove: [params.data] }); // Удаляем пользователя с помощью applyTransaction
        this.logs.push(`Удален пользователь "${removedUser.name.first}"`);
      }
    }
  },
};
</script>
