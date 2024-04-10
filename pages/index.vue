<template lang="pug">
div
  h1 Gestor de tareas
  v-row(justify="center" align="center")
    v-layout(row wrap pa-6)
      v-flex(xs12 sm12 md12 offset-md0 pa-5)
          v-card(style="max-height: auto; padding: 0px;" class="scroll-y" elevation="1" shaped)
              v-card-title
                  v-dialog(v-model="dialog" max-width="500px" height="auto")
                      template(v-slot:activator="{on}")
                          v-btn(v-on="on" color="error" type="filled" style="height:40px; margin-bottom:10px; margin-right: 15px;") Nueva tarea

                      //- Agregamos la modal 
                      v-card
                          v-card-title
                              span(class="headline")
                              h3 {{ formTitle }} (*) Obligatorios
                              v-container(grid-list-md)
                                v-layout(row wrap)
                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.title" label="* Título" required)
                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.is_completed" type="number" label="* No completada(0) Completada(1)" required)
                                  
                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.due_date" label="* Fecha (YYYY-MM-DD)" )
                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.comments" label="Comentarios")

                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.description" label="Descripción")
                                  v-flex(xs12 sm12 md12)
                                    v-text-field(v-model="editedItem.tags" label="Tags")
                                      
                          
                          v-card-actions
                              v-spacer
                              v-btn(color="error" @click.native="close") Cancel
                              v-btn(v-if="editedIndex == -1" color="primary" text-color="white" @click.native="guardarTarea") Guardar
                              v-btn(v-else color="cyan" text-color="white" @click.native="editarTarea") Editar
                  
                  v-spacer
                  v-text-field(v-model="search" append-icon="mdi-feature-search-outline" label="Buscar" solo single-line)

          //- Creamos la tabla
          v-data-table(:headers="headers"
                :items="task"
                :search="search"
                :items-per-page="10"
                :page="10"
                class="elevation-1")
              
              //- Agregamos los botones de editar y eliminar
              template(v-slot:item.action="{ item }")
                v-icon(class="mr-2 cursor-pointer" color="cyan" @click="editItemTarea(item)") mdi-border-color
                v-icon(class="cursor-pointer" color="red" @click="elimiarTarea(item.id)") mdi-delete
              
              //- Si es cero la tarea esta pendiente y 1 esta completa
              template(v-slot:item.is_completed="{ item }") 
                v-chip(v-if="item.is_completed == '0'" class="ma-2" color="red" text-color="white") Pendiente
                v-chip(v-else class="ma-2" color="green" text-color="white") Completado
  
  //- Agregamos la notificacion
  v-snackbar(v-model="snackbar") {{ text }}
    template(v-slot:action="{ attrs }")
      v-btn(color="green" text v-bind="attrs" @click="snackbar = false") Close

</template>

<script>

export default {
  name: 'Inicio',
  head: {
    titleTemplate: 'Inicio - Gestor de tareas'
  },
  data(){
    return{
      token: "e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd",
      headers_fetch: {
        'Authorization': `Bearer e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd`,
        'Content-Type': 'application/json'
      },
      task: [],

      search: '',
      headers: [
        { text: 'No.', value: 'cont'},
        { text: 'Titulo', value: 'title', align: 'left' },
        { text: 'Fecha', value: 'due_date', align: 'left' },
        { text: 'Completado', value: 'is_completed', align: 'left' },
        { text: 'Acciones', value:'action', sortable: false}
      ],

      snackbar: false,
      text: '',
      dialog: false,
      editedIndex: -1,
      id: '',
      editedItem: {
        token: '',
        title: '',
        is_completed: 0,
        due_date: '',
        comments: '',
        description: '',
        tags: ''
      },
      defaultItem: {
        token: '',
        title: '',
        is_completed: 0,
        due_date: '',
        comments: '',
        description: '',
        tags: ''
      }
    }
  },

  watch:{
    dialog(val){
      val || this.close()
    }
  },

  async created(){
    // Obtenemos los datos
    this.obtenerTareas()
  },

  computed: {
    formTitle(){
      return this.editedIndex === -1 ? 'Nueva tarea' : 'Editar tarea'
    }
  },

  methods:{
    //Creamos el metodo para obtener todas las tareas
    async obtenerTareas(){

      const getDatos = await fetch(`https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks?token=${this.token}`,
      {
        method: 'GET',
        headers: { 'Authorization': `Bearer ${this.token}`}
      })
      // Obtenemos los datos
      const res = await getDatos.json()
      //Los pasamos a las tareas
      this.task = res
      //Agregamos el indice
      for(let x in res){
        this.task[x].cont = x
      }
    },

    //Funcion para editar una tarea
    async editItemTarea(item){
      //Cambiamos el valor del texto de la modal(1 = Editar tarea)
      this.editedIndex = 1

      const getDatos = await fetch(`https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${item.id}?token=${this.token}`,
      {
        method: 'GET',
        headers: { 'Authorization': `Bearer ${this.token}`}
      })
      //Obtenemos los datos
      const  res = await getDatos.json()

      //Validamos que traega informacion
      if(res.length > 0){
        // Pasamos los datos al editedItem
        this.id = item.id
        this.editedItem.token = this.token
        this.editedItem.title = res[0].title
        this.editedItem.is_completed = res[0].is_completed
        this.editedItem.due_date = res[0].due_date
        this.editedItem.comments = res[0].comments
        this.editedItem.description = res[0].description
        this.editedItem.tags = res[0].tags

        // Mostramos la ventana modal
        this.dialog = true
      }
    },

    // Funcion para guardar una tarea nueva
    guardarTarea(){

      //Agregamos el token al objecto
      this.editedItem.token = this.token

      //Checamos que no vengan vacios los campos de titulo, is_completed y due_date
      if(this.editedItem.title != "" && this.editedItem.due_date != "" && this.editedItem.is_completed >= 0 && this.editedItem.is_completed <= 1){
        // Hacemos la consulta
        try{
          fetch('https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks', {
              method: 'POST',
              headers: this.headers_fetch,
              body: JSON.stringify(this.editedItem)
          }).then(response => response.json())
          .then((data) => {
            // Si todo esta bien cargamos la lista de tareas
            if(data.detail == 'Created task success'){
              this.obtenerTareas()
              this.close()

              // Mostramos un modal para tarea creada
              this.text = 'Tarea creada correctamente!'
              this.snackbar = true
            }
          }).catch(error => alert("Error: " + error))

        }catch(e){
          console.log('Error: ', e)
        }
      }else{
        // Mostramos un modal para tarea creada
        this.text = 'Error, campos obligatorios vacios ó datos incorrectos!'
        this.snackbar = true
      }
    },

    // Funcion para editar  una tarea existente
    editarTarea(){
      //Agregamos el token al objecto
      this.editedItem.token = this.token
      //Checamos que no vengan vacios los campos de titulo, is_completed y due_date
      if(this.editedItem.title != "" && this.editedItem.due_date != "" && this.editedItem.is_completed >= 0 && this.editedItem.is_completed <= 1){
        // Hacemos la consulta
        try{
          fetch(`https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${this.id}`, {
              method: 'PUT',
              headers: this.headers_fetch,
              body: JSON.stringify(this.editedItem)
          }).then(response => response.json())
          .then((data) => {
            // Si todo esta bien cargamos la lista de tareas
            if(data.detail == 'Updated task success'){
              this.obtenerTareas()
              this.close()
              // Mostramos un modal para tarea creada
              this.text = 'Tarea editada correctamente!'
              this.snackbar = true
            }
          }).catch(error => alert("Error: " + error))

        }catch(e){
          console.log('Error: ', e)
        }
      }else{
        // Mostramos un modal para tarea creada
        this.text = 'Error, campos obligatorios vacios ó datos incorrectos!'
        this.snackbar = true
      }
    },

    async elimiarTarea(id){
      try{
        const deleteDatos = await fetch(`https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${id}?token=${this.token}`,
        {
          method: 'DELETE',
          headers: { 'Authorization': `Bearer ${this.token}`}
        })
        // Obtenemos los datos
        const res = await deleteDatos.json()
        // Verificamos que se haya eliminado correctamente
        if(res.detail == "Deleted task success"){
          this.obtenerTareas()
          this.close()
          // Mostramos mensaje
          this.text = 'Tarea eliminada correctamente!'
          this.snackbar = true
        }
      }catch(e){
        console.log('Error: ', e)
      }
    },

    // Funcion para cuando cerremos la modal
    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedIndex = -1
        // Reiniciamos los valores por default
        this.editedItem = Object.assign({}, this.defaultItem)
      }, 300)
    },
  }
}
</script>

<style scoped>
  .cursor-pointer{
    cursor: pointer;
  }
</style>