<script setup></script>

<template>
  <div id="app">
    <nav class="navbar bg-body-tertiary px-3">
      <div class="container-fluid">
        <a class="navbar-brand py-0" href="">
          <img src="./assets/logo.png" alt="Logo" width="50" height="50" class="d-inline-block align-text-top">
        </a>
        <button type="button" @click="mostrarReservasPasadas" class="navbar-text btn">
          {{ mostrarPasadas ? 'Registros activos' : 'Registros pasados' }}
        </button>
      </div>
    </nav>
    <div class="container-fluid ">
      <div class="row text-center my-3">
        <h1>Reservas de Laboratorio - EICT</h1>
      </div>
      <button class="btn btn-primary mb-3" data-bs-toggle="modal" data-bs-target="#createModal" ref="createModal">Agregar Reserva</button>

      <table class="table table-hover">
        <thead>
        <tr>
          <th scope="col">ID</th>
          <th scope="col">Nombre</th>
          <th scope="col">Laboratorio</th>
          <th scope="col">Fecha</th>
          <th scope="col">Acciones</th>
        </tr>
        </thead>
        <tbody v-if="mostrarPasadas">

          <tr v-for="reserva in reservasPasadas" :key="reserva.idReserva">
            <td>{{reserva.id}}</td>
            <td>{{reserva.nombre}}</td>
            <td>{{reserva.laboratorio}}</td>
            <td>{{reserva.fechaReserva}}</td>
            <td>
              <button class="btn btn-primary me-3" data-bs-toggle="modal" data-bs-target="#editModal" @click="openEditModal(reserva)">Editar</button>
              <button class="btn btn-danger" @click="removeReserva(reserva.idReserva)">Eliminar</button>
            </td>
          </tr>
          </tbody>
        <tbody v-else>

          <tr v-for="reserva in reservasActivas" :key="reserva.idReserva">
            <td>{{reserva.id}}</td>
            <td>{{reserva.nombre}}</td>
            <td>{{reserva.laboratorio}}</td>
            <td>{{ formatDate(reserva.fechaReserva) }}</td>
            <td>
              <button class="btn btn-primary me-3" data-bs-toggle="modal" data-bs-target="#editModal" @click="openEditModal(reserva)">Editar</button>
              <button class="btn btn-danger" @click="removeReserva(reserva.idReserva)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="createModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="exampleModalLabel">Agregar Reserva</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form id="crearReserva" @submit.prevent="addReserva">
              <div class="mb-3">
                <label for="id" class="form-label">ID</label>
                <input type="text" class="form-control" id="id" v-model="id" required>
              </div>
              <div class="mb-3">
                <label for="nombre" class="form-label">Nombre</label>
                <input type="text" class="form-control" id="nombre" v-model="nombre" required>
              </div>
              <div class="mb-3">
                <label for="correo" class="form-label">Email</label>
                <input type="email" class="form-control" id="correo" v-model="correo" required>
              </div>
              <div class="mb-3">
                <label for="laboratorio" class="form-label">Laboratorio</label>
                <input type="text" class="form-control" id="laboratorio" v-model="laboratorio" required>
              </div>
              <div class="mb-3">
                <label for="fecha" class="form-label">Fecha</label>
                <input type="datetime-local" class="form-control" id="fecha" v-model="fechaReserva" :min="getCurrentDateTime()" required>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" ref="Close" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
            <button type="submit" form="crearReserva" class="btn btn-primary">Agregar</button>
          </div>
        </div>
      </div>
    </div>

    <!--Editar Modal-->
    <div class="modal fade" id="editModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="exampleModalLabel">Editar Reserva</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form id="editReserva" @submit.prevent="editReserva">
              <div class="mb-3">
                <label for="id" class="form-label">ID</label>
                <input type="text" class="form-control" id="id" v-model="id" required>
              </div>
              <div class="mb-3">
                <label for="nombre" class="form-label">Nombre</label>
                <input type="text" class="form-control" id="nombre" v-model="nombre" required>
              </div>
              <div class="mb-3">
                <label for="correo" class="form-label">Email</label>
                <input type="email" class="form-control" id="correo" v-model="correo" required>
              </div>
              <div class="mb-3">
                <label for="laboratorio" class="form-label">Laboratorio</label>
                <input type="text" class="form-control" id="laboratorio" v-model="laboratorio" required>
              </div>
              <div class="mb-3">
                <label for="fecha" class="form-label">Fecha</label>
                <input type="datetime-local" class="form-control" id="fecha" v-model="fechaReserva" :min="getCurrentDateTime()" required>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" ref="closeEdit" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
            <button type="submit" form="editReserva" class="btn btn-primary">Editar</button>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>

<script>

  const apiUrl = 'https://gltewsmc6d.execute-api.us-east-1.amazonaws.com/default/practica-7';
  export default {

    data() {
      return {
        reservas: [],
        idReserva: "",
        id: "",
        nombre: "",
        laboratorio: "",
        fechaReserva: this.getCurrentDateTime(),
        email: "",
        correo: "",
        mostrarPasadas: false
      }
    },
    async created() {
      await this.fetchReservas();
    },
    computed: {
      reservasActivas() {
        const ahora = new Date();
        return this.reservas.filter(reserva => new Date(reserva.fechaReserva) > ahora);
      },
      reservasPasadas() {
        const ahora = new Date();
        return this.reservas.filter(reserva => new Date(reserva.fechaReserva) <= ahora);
      }
    },
    methods: {
      async fetchReservas() {
        try {
          const response = await fetch(apiUrl);
          const jsonResponse = await response.json();
          this.reservas = jsonResponse.reservas;
        } catch (error) {
          console.error('Error fetching reservations:', error);
        }
      },
      async addReserva() {
        let reserva = {
            id: this.id,
            nombre: this.nombre,
            correo: this.correo,
            laboratorio: this.laboratorio,
            fechaReserva: this.fechaReserva
        }

        // this.reservas.push(reserva);
        await fetch(apiUrl, {
          mode: 'no-cors',
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(reserva)
        });

        await this.fetchReservas();

        this.id = ""
        this.nombre = ""
        this.correo = ""
        this.laboratorio = ""
        this.fechaReserva = this.getCurrentDateTime();
        this.$refs.Close.click();
      },
      async removeReserva(idReserva) {
        const reserva = this.reservas.find(reserva => reserva.idReserva === idReserva);

        await fetch(apiUrl, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(reserva)
        });

        await this.fetchReservas();
      },
      async editReserva() {
        let reserva = {
          idReserva: this.idReserva,
          id: this.id,
          nombre: this.nombre,
          correo: this.correo,
          laboratorio: this.laboratorio,
          fechaReserva: this.fechaReserva
        };

        // this.reservas.push(reserva);
        await fetch(apiUrl, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(reserva)
        });

        await this.fetchReservas();

        this.id = ""
        this.nombre = ""
        this.correo = ""
        this.laboratorio = ""
        this.fechaReserva = this.getCurrentDateTime();
        this.$refs.closeEdit.click();
      },
      openEditModal(reserva) {

        this.idReserva = reserva.idReserva;
        this.id = reserva.id;
        this.nombre = reserva.nombre;
        this.correo = reserva.correo;
        this.laboratorio = reserva.laboratorio;
        this.fechaReserva = reserva.fechaReserva;

      },
      getCurrentDateTime() {
        const now = new Date();
        const offset = now.getTimezoneOffset();
        return new Date(now.getTime() - offset * 60 * 1000).toISOString().slice(0, 16);
      },
      mostrarReservasPasadas() {
        this.mostrarPasadas = !this.mostrarPasadas;
      },
      formatDate(fechaReserva) {
        const date = new Date(fechaReserva);
        const formattedDate = date.toLocaleString('es-DO', {
          day: '2-digit',
          month: '2-digit',
          year: 'numeric',
          hour: '2-digit',
          minute: '2-digit'
        });
        return formattedDate;
      },
    },
}
</script>

<style scoped>

</style>
