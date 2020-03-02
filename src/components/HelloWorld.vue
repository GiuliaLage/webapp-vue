<template>
  <div>
    <div class="row">
      <div class="col col-lg-2">
        <button
          @click="addCharacter"
          type="button"
          class="btn btn-dark button-margin" 
        >Adicionar</button>
      </div>
      <div v-if="showForm" class="col col-sm">
        <form @submit.prevent="SaveCharacter">
          <div class="form-group row">
            <label for="nome" class="col-sm-1 col-form-label">Nome:</label>
            <div class="col-sm-3">
              <input type="text" class="form-control" id="nome" v-model="user.name" />
            </div>
          </div>
          <div class="form-group row">
            <label for="phone" class="col-sm-1 col-form-label">Phone:</label>
            <div class="col-sm-3">
              <input type="text" class="form-control" id="phone" v-model="user.phone" />
            </div>
          </div>
          <div class="form-group row">
            <label for="profession" class="col-sm-1 col-form-label">Profession:</label>
            <div class="col-sm-3">
              <input type="text" class="form-control" id="profession" v-model="user.profession" />
            </div>
          </div>
          <div class="col-lg-10">
            <button type="submit" class="btn btn-dark button-margin col col-sm-2">Salvar</button>
          </div>
          <div class="col-lg-10">
           <button @click="closeForm" type="button" class="btn btn-dark button-margin col col-sm-2">Cancelar</button>
          </div>
        </form>
      </div>
    </div>
  <div class="container col col-lg-6"> 
    <table class="table">
      <thead class="thead-dark">
        <tr>
          <th>Name</th>
          <th>Phone</th>
          <th>Profession</th>
          <th>Editar</th>
          <th>Excluir</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="usuario in usuarios" :key="usuario.name">
          <td>{{usuario.name}}</td>
          <td>{{usuario.phone}}</td>
          <td>{{usuario.profession}}</td>
          <td>
            <button @click="SelecionarUsuario(usuario)" type="button" class="btn btn-info">
              <i class="fas fa-user-edit"></i>
            </button>
          </td>
          <td>
            <button @click="DeletarUsuario(usuario)" type="button" class="btn btn-danger">
              <i class="far fa-trash-alt"></i>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  </div>
</template>

<script>
import axios from "axios";
import config from "./../config/config";

export default {
  data() {
    return {
      usuarios: [],
      user: { name: "", phone: "", profession: ""},
      showForm: false,
      UsuarioSelecionado: false
    };
  },
  methods: {
    addCharacter() {
      this.user.name = ""
      this.user.phone = ""
      this.user.profession = ""
      this.showForm = true 
      this.UsuarioSelecionado = false;
    },
    closeForm(){

      this.showForm = false;
    },

    SaveCharacter() {
      //metodo pra salvar se não houver nenhum usuario selecionado 
      if (this.UsuarioSelecionado === false) {
        axios.post(`${config.BASE_URL}/user`, this.user).then(response => {
          this.usuarios.push(response.data);
          this.showForm = !this.showForm;
        });
      } else {
        //metodo pra editar se houver um usuario selecionado 

         axios.put(`${config.BASE_URL}/user/${this.user.id}`, this.user).then(response =>{
        const indice = this.usuarios.findIndex(u => u.id === response.data.id) // procura o index do array
        this.usuarios.splice(indice,1,this.user) // indice, quantidade, o que é pra adicionar 
        this.showForm = !this.showForm
        this.UsuarioSelecionado = false; 
        })
      }
    },
    SelecionarUsuario(usuario) {
      this.user.name = usuario.name;
      this.user.phone = usuario.phone;
      this.user.profession = usuario.profession;
      this.user.id = usuario.id; 
      this.UsuarioSelecionado = true;
      this.showForm = true;
    },
    DeletarUsuario(usuario){
      this.user.id = usuario.id
      this.user.name = usuario.name 

      const confirmacao = window.confirm(`Deseja realmente excluir usuário ${this.user.name}`)
      if(confirmacao){
      
        axios.delete(`${config.BASE_URL}/user/${this.user.id}` , this.user).then(response =>{
      const indice = this.usuarios.findIndex(u => u.id === response.data.id) // procura o index do array
      this.usuarios.splice(indice,1) // indice, quantidade, o que é pra adicionar 
      })

      }

      
    }
  },

  watch: {
    "user.name"(newUser) {
      this.user.name = newUser;
    }
  },

  created() {
    axios.get(`${config.BASE_URL}/user`).then(response => {
      this.usuarios = response.data;
    });
  }
};
</script>

<style>
.button-margin {
  margin-bottom: 20px;
}
</style>