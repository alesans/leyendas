<template>
  <div>
      <form @submit="editarLibro(libro)" v-if='activo'>
        <h3 class="text-center mb-4">Edición de leyenda</h3>
        <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="libro.nombre">
        <input type="text" placeholder="Provincia" class="form-control mb-2" v-model="libro.provincia">
        <textarea class="form-control mb-3" id="exampleFormControlTextarea1" rows="3" placeholder="Descripción" v-model="libro.descripcion"></textarea>
        <button class="btn form-control mb-2 btn-success" type="submit">Modificar</button>
      </form>

       <form @submit.prevent="agregar" v-else>
        <h3 class="text-center mb-4">Registro de leyenda</h3>
        <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="libro.nombre">
        <input type="text" placeholder="Provincia" class="form-control mb-2" v-model="libro.provincia">
        <textarea class="form-control mb-3" id="exampleFormControlTextarea1" rows="3" placeholder="Descripción" v-model="libro.descripcion"></textarea>
        <button class="btn form-control mb-2 btn-primary" type="submit">Registrar</button>
      </form>

      <div class="row">
        <input class="form-control my-4" type="text" v-model="buscar" placeholder="buscar por nombre">
        <div class="card col-sm-1 col-md-6"
        v-for="(item,index) in filtro" :key="index">
        <div class="py-5 m-auto" >
          <div class="text-center">
              <p class="w-auto">{{item.nombre}}</p>
              <p class="w-auto">{{item.provincia}}</p>
          </div>
          <div>
              <p class="w-auto">{{item.descripcion}}</p>
          </div>
          <div class="text-center">
            <button class="btn btn-dark btn-sm" @click='editar(item)'>Modificar</button>
            <button class="btn btn-danger btn-sm" @click='eliminar(item,index)'>Eliminar</button>
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
      libros: [],
      libro: {nombre: '', provincia: '', descripcion: ''},
      buscar: '',
      activo: false,
    }
  },
  created() {
    axios.get('/libros')
    .then(res => {
      this.libros = res.data;
    })
  },
  methods: {
    agregar(){
      if(this.libro.nombre.trim() === '' || this.libro.provincia.trim() === '' || this.libro.descripcion.trim() === ''){
        alert('Por favor llenar los campos ');
        return;
      }
      console.log(this.libro.nombre, this.libro.provincia, this.libro.descripcion);
      const params = {
        nombre: this.libro.nombre, 
        provincia: this.libro.provincia, 
        descripcion: this.libro.descripcion
      }
      this.libro.nombre = '';
      this.libro.provincia = '';
      this.libro.descripcion = ''; 
      axios.post('/libros', params)
      .then(res => {
        this.libros.push(res.data)
      })     
    },
    editar(item){
      this.libro.id = item.id;
      this.libro.nombre = item.nombre;
      this.libro.provincia = item.provincia;
      this.libro.descripcion = item.descripcion;
      this.activo = true;
    },
    editarLibro(item){
      const params = {
        nombre: item.nombre, 
        provincia: item.provincia, 
        descripcion: item.descripcion
        }
      this.libro.nombre = '';
      this.libro.provincia = '';
      this.libro.descripcion = ''; 
      axios.put(`/libros/${item.id}`, params)
        .then(res=>{
          this.activo = false;
          const index = this.libros.findIndex(
            libroFind => libroFind.id === res.data.id)
            this.libros[index] = res.data;
        })
    },
    eliminar(item,index){
      const confirmacion = confirm(`¿Está usted seguro de eliminar la leyenda? ${item.nombre}`);

      if(confirmacion) {
          axios.delete(`/libros/${item.id}`)
          .then(()=>{
          this.libros.splice(index, 1);
        })
      }
    }
  },
  computed: {
    filtro() {
      return this.libros.filter((libro)=> {
        return libro.nombre.toLowerCase().match(this.buscar.toLowerCase());
      })
    }
  }
}
</script>