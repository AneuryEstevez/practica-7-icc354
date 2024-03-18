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
            <td>{{reserva.fecha}}</td>

          </tr>
          </tbody>
        <tbody v-else>

          <tr v-for="reserva in reservasActivas" :key="reserva.idReserva">
            <td>{{reserva.id}}</td>
            <td>{{reserva.nombre}}</td>
            <td>{{reserva.laboratorio}}</td>
            <td>{{reserva.fecha}}</td>
            <td>
              <button class="btn btn-primary me-3" @click="openEditModal(reserva)">Editar</button>
              <button class="btn btn-danger" @click="removeReserva(reserva.id)">Eliminar</button>
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
                <input type="datetime-local" class="form-control" id="fecha" v-model="fecha" :min="getCurrentDateTime()" required>
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
  </div>

</template>

<script>
  export default {

    data() {
      return {
        reservas: [],
        id: "",
        nombre: "",
        laboratorio: "",
        fecha: this.getCurrentDateTime(),
        email: "",
        correo: "",
        mostrarPasadas: false
      }
    },
    async created() {
      let response = await fetch('https://0x2l3y6r90.execute-api.us-east-1.amazonaws.com/default/myFuncion')
      response = await response.json();
      this.reservas = response.reservas;

      console.log(this.reservas)

    },
    computed: {
      reservasActivas() {
        const ahora = new Date();
        return this.reservas.filter(reserva => new Date(reserva.fecha) > ahora);
      },
      reservasPasadas() {
        const ahora = new Date();
        return this.reservas.filter(reserva => new Date(reserva.fecha) <= ahora);
      }
    },
    methods: {
      addReserva() {

        // Find the reservation with the matching ID
        const index = this.reservas.findIndex(reserva => reserva.id === this.id);
        let reserva;

        if (index > -1) {
          reserva = this.reservas.splice(index, 1, {
            id: this.id,
            nombre: this.nombre,
            correo: this.correo,
            laboratorio: this.laboratorio,
            fecha: this.fecha
          });
        } else {
          reserva = {
            id: this.id,
            nombre: this.nombre,
            correo: this.correo,
            laboratorio: this.laboratorio,
            fecha: this.fecha
          }
        }

        this.reservas.push(reserva);

        this.id = ""
        this.nombre = ""
        this.correo = ""
        this.laboratorio = ""
        this.fecha = this.getCurrentDateTime();
        this.$refs.Close.click();
      },
      removeReserva(id) {
        this.reservas = this.reservas.filter(reserva => reserva.id !== id)
      },
      openEditModal(reserva) {
        // Populate the form fields with the selected reservation data
        this.id = reserva.id;
        this.nombre = reserva.nombre;
        this.correo = reserva.correo; // Assuming 'correo' is the desired email field
        this.laboratorio = reserva.laboratorio;
        this.fecha = reserva.fecha;

        this.$refs.createModal.click();

        // this.id = ""
        // this.nombre = ""
        // this.correo = ""
        // this.laboratorio = ""
        // this.fecha = this.getCurrentDateTime();
        // this.$refs.Close.click();
      },
      getCurrentDateTime() {
        const now = new Date();
        const offset = now.getTimezoneOffset();
        return new Date(now.getTime() - offset * 60 * 1000).toISOString().slice(0, 16);
      },
      async getReservas() {
        const response = await fetch('http://localhost:3000/reservas')
        this.reservas = await response.json()
      },
      mostrarReservasPasadas() {
        this.mostrarPasadas = !this.mostrarPasadas;
      }
    },
}
</script>

<style scoped>

</style>
