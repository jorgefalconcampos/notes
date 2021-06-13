<template>
<div class="container">
    <b-alert
    :show="dismissCountDown"
    dismissible
    :variant="mensaje.color"
    @dismissed="dismissCountDown=0"
    @dismiss-count-down="countDownChanged"
    > {{mensaje.texto}}
    </b-alert>


    <!-- editarNota es el nombre del formulario -->
    <form @submit.prevent="editarNota(notaEditar)" v-if="editar">  
        <h3> Editar nota </h3>
        <input type="text" class="form-control my-2" placeholder="Nombre" v-model="notaEditar.nombre">
        <input type="text" class="form-control my-2" placeholder="Descripción" v-model="notaEditar.descripcion">
        <b-button class="btn-warning my-2 mx-1" type="submit"> Editar nota </b-button>
        <b-button class="my-2 mx-1" type="submit" @click="editar=false"> Cancelar edición </b-button>
    </form>



    <!-- agregarNota es el nombre del formulario -->
    <form @submit.prevent="agregarNota()" v-if="!editar"> 
        <h3> Agregar nueva nota </h3>
        <input type="text" class="form-control my-2" placeholder="Nombre" v-model="nota.nombre">
        <input type="text" class="form-control my-2" placeholder="Descripción" v-model="nota.descripcion">
        <b-button class="btn-success my-2" type="submit"> Agregar nota </b-button>

    </form>


    <table class="table">
        <thead>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Nombre</th>
                <th scope="col">Descripción</th>
                <th scope="col">Fecha</th>
                <th scope="col">Acciones</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(item, index) in notas" :key="index">
                <th scope="row">{{item._id}}</th>
                <td>{{item.nombre}}</td>
                <td>{{item.descripcion}}</td>
                <td>{{item.date}}</td>
                <td>
                <b-button class="btn-warning btn-sm mx-2" @click="activarEdicion(item._id)">Editar</b-button>
                <b-button class="btn-danger btn-sm mx-2" @click="eliminarNota(item._id)">Eliminar</b-button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
</template>


<script>
export default {
  data() {
    return {
      notas: [],
       mensaje: {color: 'success', texto: ''},
      dismissSecs: 5,
      dismissCountDown: 0,
      nota: { nombre: '', descripcion: ''},
      editar: false,
      notaEditar: { }
    };
  },
  created(){
    this.listarNotas();
  },
  methods:{
    listarNotas(){
      this.axios.get('/nota')
      .then((response) => {
        // console.log(response.data)
        this.notas = response.data;
      })
      .catch((e)=>{
        console.log('error' + e);
      })
    },
    agregarNota(){
    //   console.log(this.nota);  
      this.axios.post('/new-note', this.nota)
      .then(res => {
          this.notas.push(res.data)
          this.nota.nombre='';
          this.nota.descripcion = '';
          this.mensaje.color = 'success';
          this.mensaje.texto = 'Nota agregada';
          this.showAlert()
      })
      .catch(e => {
        //   console.log(e.response);
          if(e.response.data.error.errors.nombre.mensaje){
              this.mensaje.texto = e.response.data.error.errors.nombre.mensaje
          }
          else{
              this.mensaje.texto = 'Error de sistema';
          }
           this.mensaje.color = 'danger';
          this.mensaje.texto = 'Nombre obligatorio';
          this.showAlert()
      })
    },

    eliminarNota(id){
      console.log(id);
      this.axios.delete(`/nota/${id}`)
      .then(res => {
        const index = this.notas.findIndex(item => item._id === res.data._id)
        this.notas.splice(index, 1);
        this.mensaje.color = 'success';
        this.mensaje.texto = "Nota eliminada";
        this.showAlert();
      })
      .catch(e => {
        console.log(e.response);
      })
    },
    
    activarEdicion(id){
      this.editar=true;
      console.log(id);
      this.axios.get(`/nota/${id}`)
      .then(res => {
        this.notaEditar = res.data;
      })
      .catch(e => {
        console.log(e.response);
      })
    },

    editarNota(item){
      this.axios.put(`/nota/${item._id}`, item)
      .then(res => {
        const index = this.notas.findIndex(n => n._id === res.data._id);
        this.notas[index].nombre = res.data.nombre;
        this.notas[index].descripcion = res.data.descripcion;
        this.mensaje.color = 'success';
        this.mensaje.texto = "Nota actualizada";
        this.showAlert();
        this.editar=false;
      })
      .catch(e => {
        console.log(e.response);
      })

    },
     countDownChanged(dismissCountDown) {
      this.dismissCountDown = dismissCountDown
    },
    showAlert() {
      this.dismissCountDown = this.dismissSecs
    },
  },

};

</script>


