<template>
  <a class="fixo button is-large is-danger is-loading" v-show="isLoading">Carregando</a>
  <div class="container">
    <h1 class="title">{{title}}</h1>
    <div class="columns">
      <div class="column is-5">
        <p class="control has-addons">
          <input class="input is-expanded" type="text" placeholder="Procure pelo nome" v-model="search">
          <a class="button is-info" @click.prevent="searchContato">Procurar</a>
        </p>
      </div>

    <div class="column is-6"></div>
    <div class="column is-1">
     <a class="button is-info" @click.prevent="newContato">Novo</a>
    </div>
  </div>
    <div class="columns">
      <div class="column is-12">
        <table class="table is-narrow is-bordered">
          <thead>
            <tr>
              <th>Nome</th>
              <th>Telefone</th>
              <th>Email</th>
              <th>Ações</th>
            </tr>
        </thead>
        <tbody>
          <tr v-for="contato in contatos">
            <td>{{contato.nome}}</td>
            <td>{{contato.telefone}}</td>
            <td>{{contato.email}}</td>
            <td class="is-icon">
              <a href="#" @click.prevent="editContato(contato)">
                <i class="fa fa-edit"></i>
              </a>
              <a href="#" @click.prevent="removeContato(contato)">
                <i class="fa fa-trash"></i>
              </a>
            </td>
          </tr>
        </tbody>
      </table>
      <Pagination :total="total" :page="page" :itens-per-page="itensPerPage" @change-page="onChangePage"></Pagination>
    </div>
  </div>
</div>

<div id="modal_contato" class="modal" :class="{'is-active':showModal}">
  <div class="modal-background"></div>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Contato: {{selected.nome}}</p>
      <button class="delete" @click.prevent="showModal=false"></button>
    </header>
    <section class="modal-card-body">
      <div class="columns">
        <div class="column">
          <label class="label">Código</label>
          <p class="control">
            <input class="input" type="text" placeholder="Código" v-model="selected.codigo">
          </p>
          <label class="label">Nome</label>
          <p class="control">
            <input class="input" type="text" placeholder="Seu nome" v-model="selected.nome">
          </p>
          <label class="label">Telefone</label>
          <p class="control">
            <input class="input" type="phone" placeholder="(xx) 98888-5555" v-model="selected.telefone">
          </p>
          <label class="label">Email</label>
          <p class="control">
            <input class="input" type="email" placeholder="Seu email" v-model="selected.email">
          </p>
        </div>
      </div>
    </section>
    <footer class="modal-card-foot">
      <a class="button is-primary" @click.prevent="saveContato">Salvar</a>
      <a class="button" @click.prevent="showModal=false">Cancelar</a>
    </footer>
  </div>
</div>
</template>

<script>
  import Pagination from './Pagination.vue'
  
  export default {
    data () {
      return {
        isLoading: false,
        title: 'Vue.js Crud',
        search: '',
        contatos: [],
        page: 1,
        total: 0,
        selected: {},
        itensPerPage: 10,
        showModal:false
      }
    },
    components: {
      Pagination
    },
    methods: {
      onChangePage(page){
        this.page = page
        this.loadContatos()
      },
      showLoading(){
        this.isLoading=true;
      },
      hideLoading(){
        this.isLoading=false;
      },
      loadContatos(){

        let t = this
        this.showLoading()

        let start = (this.page * this.itensPerPage) - (this.itensPerPage)
        let end = this.page * this.itensPerPage
        let qString = "";

        if (this.search){
          qString = `&q=${this.search}`
        }

        this.$http.get(`/contatos?_start=${start}&_end=${end}${qString}`).then(
         response=>{
           t.contatos = response.json()
           t.total = response.headers['x-total-count']
         },
         error=>{
           console.log(error)
         }).finally(function () {
          t.hideLoading();
        })

       },
       searchContato(){
        this.loadContatos()
       },
       newContato(){
        this.selected={}
        this.showModal = true;
       },
       editContato(contato){
        this.selected=contato
        this.showModal = true;
       },
       removeContato(contato){
        let self = this;
        swal({  title: "Você tem certeza?",
                 text: `Deseja apagar "${contato.nome}"`,   
                 type: "warning",   
                 showCancelButton: true,   
                 confirmButtonColor: "#DD6B55",   
                 cancelButtonText: "Cancelar",
                 confirmButtonText: "Sim, pode apagar!", 
                 showLoaderOnConfirm: true,  
                 closeOnConfirm: false }, function(){   
                  
                  self.$http.delete(`/contatos/${contato.codigo}`).then(
                    result=>{
                      swal("Contato removido!")
                      self.loadContatos()
                    })
        });

       },
       saveContato(){
        if (this.selected.id!=null){  //EDIT
          this.$http.put(`/contatos/${this.selected.codigo}`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(
              this.loadContatos()
            )
          }
          else
          { //NEW
            this.$http.post(`/contatos`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(
              this.loadContatos()
            )
          }
       }
     },
     created(){
      this.loadContatos();
    },
  }
</script>
<style>
  .fixo{float: right; margin-right: 10px; margin-top: 0px; z-index: 1000;}
</style>