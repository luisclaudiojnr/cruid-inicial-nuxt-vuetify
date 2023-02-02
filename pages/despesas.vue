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
  <TableCruid :load="load" :headers="headers" :objs="despesas" title="Despesas" textoBtn="Despesa"
      @saveObject="salvar"
      @editObject="editar"
      @deletObject="deletar"> </TableCruid>
</div>
</template>


<script>


export default {
    
    data: () =>({   
      headers: [        
        { text: 'Data', value: 'data', type:'Date' },  
        { text: 'Descrição', value: 'nome', type:'String' },   
        { text: 'Categoria', value: 'categorias', type:'Object' }, 
        { text: 'Valor', value: 'valor', type:'Money' }, 
        { text: 'Ações', value: 'actions', sortable: false },
      ],
      despesas:[],
    //   categoria:{
    //     id:Number,
    //     nome: String,
    //     formaVenda: String,
    //     preco: Number
    //   },
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
        this.$axios('despesas.json').then(res =>{
          if(res.data){
            for(let chave in res.data){
              this.despesas.unshift({
                id:chave,
                data:res.data[chave].data,
                nome:res.data[chave].nome,
                categorias:res.data[chave].categorias,
                valor:res.data[chave].valor
              })
            }
          }

          this.load=false
        })
        
      },

      getObjSecundario(id, objs){  
        return Object.values(objs)[0].find(categ => categ.id === id )
      },

      salvar(obj, objsecundario){ 
        this.load=true
        this.$axios.post('despesas.json',{ 
            data: obj.data, 
            nome: obj.nome, 
            categorias: this.getObjSecundario(obj.categorias, objsecundario),
            valor:obj.valor })
          .then(res =>{
            this.despesas.unshift({
              id:res.data.name,
              data:JSON.parse(res.config.data).data,
              nome:JSON.parse(res.config.data).nome,              
              categorias:JSON.parse(res.config.data).categorias.nome,
              valor:JSON.parse(res.config.data).valor, 
            })
            this.load=false
            this.mensagem("Despesa adicionada!!")
          })
          .catch(err => this.mensagem(`Erro: ${err} ao tentar adicionar despesa.`))
        
        
      },
      editar(objeto, objSecundarios){
        this.load = true
        this.despesas.forEach(obj => {
          if(obj.id === objeto.id){
            Object.keys(obj).map(chave => {
                if(chave === 'categorias')
                  obj['categorias'] = this.getObjSecundario(objeto.categorias, objSecundarios)
                else
                 obj[`${chave}`] = objeto[`${chave}`]
            })
            this.$axios.patch(`/despesas/${obj.id}.json`, obj)
              .then(() => {
                this.load=false
                this.mensagem(`Produto alterado com sucesso!!`)
              })
              .catch(err => {
                this.load=false
                this.mensagem`Error: ${err} ao alterar produto!`
              })
          }
        })
        
      },
      deletar(id){
        this.load=true //durante o load, preciso desabilitar a tabela

        const indice = this.despesas.indexOf(this.despesas.filter(el => el.id == id)[0])
        this.$axios.delete(`/despesas/${id}.json`).then(() => {
          this.mensagem(`Despesa "${this.despesas[indice].nome}" excluida`)          
          this.despesas.splice(indice, 1)
          this.load=false
        }).catch(err => this.mensagem(`Erro ${err} ao tentar excluir despesa`))

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