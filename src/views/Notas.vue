<template>
  <div class="container">
    <h1>Notas</h1>
    <b-alert
      :show="dissmissCountDown" 
      dismisable
      :variant="mensaje.color"
      @dismissed="dissmissCountDown=0"
      @dismiss-count-down="countDownChange">
      {{mensaje.text}}
    </b-alert>

    <form @submit.prevent="agregarNota()" v-if="agregar">
      <h3 class="text-center">Agregar Nueva Nota</h3>
      <input type="text" required placeholder="Ingresa el nombre" class="form-control my-2" v-model="nota.nombre">
      <input type="text" required placeholder="Ingresa la descripción" class="form-control my-2" v-model="nota.descripcion">
      <button class="btn-sm btn-block btn-success" type="submit">Agregar</button>
    </form>

    <form @submit.prevent="editarNota()" v-else>
      <h3 class="text-center">Editar Nueva Nota</h3>
      <input type="text" required placeholder="Actualiza el nombre" class="form-control my-2" v-model="nota.nombre">
      <input type="text" required placeholder="Actualiza la descripción" class="form-control my-2" v-model="nota.descripcion">
      <button class="btn-warning my-2 mx-2" type="submit">Confirmar</button>
      <button class="btn-danger my-2 mx-2" type="submit" @click="nota={};agregar=true;">Cancelar</button>
    </form>

    <table class="table">
      <thead>
        <th scope="col">#</th>
        <th scope="col">Nombre</th>
        <th scope="col">Descripción</th>
        <th scope="col">Acciones</th>
      </thead>
      <tbody>
        <tr v-for="(item, index) in notas" :key="index">
          <td scope="row">{{item._id}}</td>
          <td>{{item.nombre}}</td>
          <td>{{item.descripcion}}</td>
          <td>
            <b-button class="btn-warning btn-sm mx-2" 
                      @click="activarEdicion(item)">Actualizar</b-button>
                    <b-button class="btn-danger btn-sm mx-2" 
                              @click="eliminarNota(item._id)">Eliminar</b-button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'Notas',
  data: () => ({
    notas: [],
    mensaje: {
      color: 'success',
      text: ''
    },
    dismissSecs: 5,
    dissmissCountDown: 0,
    agregar: true,
    nota: {},
  }),
  methods: {
    listarNotas() {
      this.axios.get('nota').then((response) => {
        this.notas = response.data;
      }).catch((e) => {
        this.mensaje.text = e;
        this.mensaje.color = 'danger';
        this.showAlert();
      })
    },
    agregarNota() {
      this.axios.post('nueva-nota', this.nota).then((response) => {
        this.notas.push(response.data);
        this.mensaje.text = 'Nota Agregada';
        this.mensaje.color = 'success';
        this.showAlert();
      }).catch((e) => {
        this.mensaje.text = e;
        this.mensaje.color = 'danger';
        this.showAlert();
      });
      this.nota = {};
    },
    eliminarNota(id) {
      this.axios.delete(`nota/${id}`).then((response) => {
        const index = this.notas.findIndex(item => item._id === response.data._id);
        this.notas.splice(index, 1);
        this.mensaje.text = 'Nota Eliminada';
        this.mensaje.color = 'success';
        this.showAlert();
      }).catch((e) => {
        this.mensaje.text = e;
        this.mensaje.color = 'danger';
        this.showAlert();
      })
    },
    activarEdicion(item) {
      this.agregar = false;
      this.nota._id = item._id;
      this.nota.nombre = item.nombre;
      this.nota.descripcion = item.descripcion;
    },
    editarNota() {
      this.axios.put(`nota/${this.nota._id}`, this.nota).then((response) => {
        const index = this.notas.findIndex(item => item._id === response.data._id);
        this.notas[index].nombre = response.data.nombre;
        this.notas[index].descripcion = response.data.descripcion;
        this.mensaje.text = 'Nota Actualizada';
        this.mensaje.color = 'success';
        this.showAlert();
      }).catch((e) => {
        this.mensaje.text = e;
        this.mensaje.color = 'danger';
        this.showAlert();
      });
      this.nota = {};
      this.agregar = true;
    },
    countDownChange(dissmissCountDown) {
      this.dissmissCountDown = dissmissCountDown;
    },
    showAlert() {
      this.dissmissCountDown = this.dismissSecs;
    }
  },
  created() {
    this.listarNotas();
  },
}
</script>
