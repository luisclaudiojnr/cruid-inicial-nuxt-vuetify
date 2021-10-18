<template>
  <div>
  <v-snackbar
      v-model="msgLog"
      :timeout="timeout"
      top
      transition="scale-transition"
    >
      {{ text }}
    </v-snackbar>
<!-- :vendas sera = itemsTable -->
  <TableCommon :load="load" :headers="headers" :objs="categorias" title="Categorias" textoBtn="Categoria"
      @saveObject="salvarCategoria"
      @editObject="editarCategoria"
      @deletObject="deletarCategoria"> </TableCommon>
</div>
</template>


<script>


export default {
    
    data: () =>({   
      headers: [        
        { text: 'Nome', value: 'nome', type:'String' },   
        { text: 'Ações', value: 'actions', sortable: false },
      ],
      categorias:[],
      categoria:{
        id:Number,
        nome: String,
      },
      load:true,
      msgLog:false,
      text:"",
      timeout:3000

      
    }),
    created() {
      this.inicialized()
    },
    methods:{
      inicialized(){ 
        this.$axios('categorias.json').then(res =>{
          if(res.data){
            for(let chave in res.data){
              this.categorias.unshift({
                id:chave,
                nome:res.data[chave].nome
              })
            }
          }

          this.load=false
        })
        
      },
      // getCategoria(id){        
      //   return this.categorias.find(categ => categ.id === id)
      // },
      salvarCategoria(categoria){ 
        this.load=true
        
        this.$axios.post('categorias.json',{ nome: categoria.nome})
          .then(res =>{
            this.categorias.unshift({
              id:res.data.name,
              nome:JSON.parse(res.config.data).nome
            })
            this.load=false
            this.mensagem("Categoria adicionada!!")
          })
          .catch(err => this.mensagem(`Erro: ${err} ao tentar adicionar categoria de despesa.`))
        
        
      },
      editarCategoria(categoria){
        this.load = true
        this.categorias.forEach(cate => {
          if(cate.id === categoria.id){
            Object.keys(cate).map(chave => {
                cate[`${chave}`] = categoria[`${chave}`]
            })
            this.$axios.patch(`/categorias/${cate.id}.json`, cate)
              .then(() => {
                this.load=false
                this.mensagem(`Categoria editada com sucesso!!`)
              })
              .catch(err => {
                this.load=false
                this.mensagem`Error: ${err} ao editar categoria!`
              })
          }
        })
        
      },
      deletarCategoria(id){
        this.load=true //durante o load, preciso desabilitar a tabela

        const indice = this.categorias.indexOf(this.categorias.filter(el => el.id == id)[0])
        this.$axios.delete(`/categorias/${id}.json`).then(() => {
          this.mensagem(`Categoria "${this.categorias[indice].nome}" excluida`)          
          this.categorias.splice(indice, 1)
          this.load=false
        }).catch(err => this.mensagem(`Erro ${err} ao tentar excluir categoria de despesa`))

      },
      mensagem(texto){
        this.msgLog = true 
        this.text = texto
        this.load = false        
      }
    }
}
</script>

<style>

</style>