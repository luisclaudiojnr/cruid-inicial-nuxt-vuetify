<template>
  <v-dialog v-model="dialog" max-width="800px" persistent>
    <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark class="mb-2"
          v-bind="attrs"
          v-on="on" >
            Adicionar {{textoBtn}}
        </v-btn>
    </template>
    <v-card>
        <v-card-title>
            <span class="headline">{{ !dialogEdit ? 'Adicionar' : 'Editar' }}  {{textoBtn}} </span>
        </v-card-title>
        <v-card-text>
          <v-form ref="form"> 
            <v-container>
              <v-row >

                <v-col v-for="campo in campos" :key="campo.value" cols="12" sm="6" md="4">

                  <v-menu v-if="campo.type==='Date'" :ref="`${campo.value}`" v-model="campo[`${campo.value}`]" :close-on-content-click="false" :return-value.sync="objeto[`${campo.value}`]"
                    transition="scale-transition" offset-y min-width="auto" >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field v-model="camposData[`${campo.value}`]" :label="campo.text" prepend-icon="mdi-calendar"
                        readonly v-bind="attrs" v-on="on" ></v-text-field>
                    </template>
                    <v-date-picker v-model="objeto[`${campo.value}`]" no-title scrollable >
                      <v-spacer></v-spacer>
                      <v-btn text color="primary" @click="campo[`${campo.value}`] = false" >
                        Cancelar
                      </v-btn>
                      <v-btn text color="primary" @click="saveDate(objeto[`${campo.value}`], `${campo.value}`)" >
                        OK
                      </v-btn>
                    </v-date-picker>
                  </v-menu>

                  <v-text-field v-if="campo.type==='String'" :label="campo.text" v-model="objeto[`${campo.value}`]" 
                      :rules="[v => !!v && !!v.trim() || `${campo.text} é obrigatorio`]" ></v-text-field>

                  <v-text-field v-if="campo.type==='Number'" :label="campo.text" type="number" 
                      :rules="[v => (v && parseInt(v) > 0) || `${campo.text} é obrigatorio`]"
                      v-model="objeto[`${campo.value}`]"  ></v-text-field>

                  <v-text-field v-if="campo.type==='Money'" :label="campo.text" type="number" prefix="R$ "
                      :rules="[v => (v && parseInt(v) > 0) || `${campo.text} deve ser maior que R$ 0,00`]"
                      v-model="objeto[`${campo.value}`]" 
                      ></v-text-field>                  

                  <v-autocomplete v-if="isObject(campo)" :items="fieldsObjets[`${campo.value}`]" item-text="nome" 
                    item-value="id" :label="campo.text"
                    auto-select-first v-model="objeto[`${campo.value}`]"                     
                     >
                  </v-autocomplete>

                  <v-checkbox v-if="campo.type=='Bool'"
                    v-model="objeto[`${campo.value}`]"
                    :label="campo.text"
                  ></v-checkbox>

                  
                  
                </v-col>
          <!--  <v-col cols="12" sm="6" md="4">
                  <v-text-field id="valorVenda" label="Valor da venda" prefix="R$ " 
                          :rules="[ v => parseInt(v.replace(',','')) > 0 || 'Valor da venda deve ser maior que R$ 0,00' ]"
                          v-model="venda.valorVenda" 
                          v-money="money" 
                          ></v-text-field>
                </v-col> -->


              </v-row>
            </v-container>
          </v-form>
        </v-card-text>
        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="close" >Cancelar</v-btn>
            <v-btn color="blue darken-1" text @click="!dialogEdit ? save() : edit()" >Salvar</v-btn>
        </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import moment from 'moment'
import { VMoney } from 'v-money'

export default {
  props: [ 'dialogEdit', 'campos', 'title', 'textoBtn'],  
  data: () =>({
    dialog:false,   
    menu: false,
    objeto: {},
    defaultItem:{},
    venda:{},
    camposData:{},
    fieldsObjets:{},
    money: {
      decimal: ',',
      thousands: '.',      
      precision: 2,
      masked:true
    },
    weight:{
      decimal: ',',
      thousands: '.',      
      precision: 3,
      masked: true
    }
  }), 
  watch:{    
    dialogEdit(val){
      if(val){
        this.dialog = true 
        this.campos.map(campo => {
          this.objeto.id = val.id
          if(campo.value !== 'actions'){
            this.objeto[`${campo.value}`] = val[`${campo.value}`]
              //console.log(campo.value)
 
          }
          if(campo.type==='Date'){
            //console.log('campo é Date ', val[`${campo.value}`])
            this.objeto[`${campo.value}`] =  val[`${campo.value}`]//.toISOString().substr(0, 10) 
            this.camposData[`${campo.value}`] = moment(this.objeto[`${campo.value}`]).format('DD/MM/YYYY')          
          }

          // if(campo.type==='Object'){

          //   //this.fieldsObjets(val[`${campo.value}`])
          //   //this.objeto[`${campo.value}`] = val[`${campo.value}`].id
          //   console.log( val[`${campo.value}`])
          // }
        })       
      }
    }   
  },
  created(){    
    this.initialize()
    this.dialog = false  

    this.campos.map( campo => {
      //console.log('campo', campo)
      if(campo.type === 'Object'){        
        this.loadObjects(campo.value)
      }
    })
  }, 
  methods:{   
    initialize(){      
      //Os itens devem estar na ordem em que chegam, para que sejam tambem renderizados ordenados na tabela
      //Primeiro item é o id que não deve ser preenchido nos campos
      this.objeto.id=''
      this.campos.map(obj =>{
        //Ordenas os objetos de acordo com headers. Ignora o actions        
        if(obj.value !== 'actions'){
          this.objeto[`${obj.value}`] = ''            
        }
        //Se data, iniciamos o campo data com a data de hoje
        if(obj.type==='Date'){
          this.objeto[`${obj.value}`] =  new Date().toISOString().substr(0, 10) 
          this.camposData[`${obj.value}`] = moment(this.objeto[`${obj.value}`]).format('DD/MM/YYYY')          
        }
      })
    },
    // openDialog(){
    //   console.log(!!this.editVenda.cliente)
    // },
    close(){      
      // document.getElementById('valorVenda').value = '0,00';
      // document.getElementById('pesoTotal').value = '0,000';
      this.$emit("dialogEdit", false)
      //Reset todos os campos com exceção dos capos tipo data
      this.$refs.form.inputs.forEach(obj =>{
        if(obj.label.toLowerCase().indexOf("data") < 0){
          obj.reset()
        }
      })
      this.initialize()      
      this.dialog = false      
    },
    save(){
      if(this.$refs.form.validate()){
        // this.venda.dataVenda = moment(this.venda.dataVenda).format('DD/MM/YYYY')
        // this.objeto.id = Math.random() * (100 - 4) + 4;         

        this.$emit("saveObject", this.objeto, this.fieldsObjets)        
        
        this.close() 
      }          
    },
    edit(){
      //console.log(this.objeto)

      if(this.$refs.form.validate()){   
        this.$emit("editObject", this.objeto, this.fieldsObjets) 
        this.close()     
      }
    },

    saveDate(item, referencia){
      this.$refs[`${referencia}`][0].save(item)
      Object.keys(this.camposData).map(obj => {
          if(obj === referencia)
            this.camposData[`${obj}`] = moment(item).format('DD/MM/YYYY')
        })      
    }, 

    isObject(campo){
      if(campo.type ==='Object'){
        //this.loadObjects(campo.value)
        return true
      }
      else {
        return false
      }
    },

    loadObjects(value){
      const temp =[]

      this.$axios(`${value}.json`).then(res =>{
        //console.log("cruid dialog",res.data)
        if(res.data){          
            for(let chave in res.data){              
              temp.unshift({
                id:chave,
                ...res.data[chave]
              })
            }
          }
      })
      const obj = {id:"001", nome:"Sem registro"}
      temp.unshift({...obj})
       
      //console.log(temp)

      //this.fieldsObjets[`${value}`] = temp
      this.fieldsObjets[`${value}`] = temp
      //console.log(this.fieldsObjets[`${value}`])
      //console.log('fieldsObject',this.fieldsObjets)      
    }
  },
  directives: {money: VMoney}
}




// formatarMoeda() {
    //     // var elemento = document.getElementById('valor');
    //     var valor = this.venda.valorVenda;
    //     // valor = valor + '';
    //     valor = parseInt(valor.replace(/[\D]+/g, ''));
    //     valor = valor + '';
    //     if(valor.length < 2)
    //       valor = valor.replace(/([0-9]{1})$/g, "0,0$1");
    //     else if(valor.length < 3)
    //       valor = valor.replace(/([0-9]{2})$/g, "0,$1");
    //     else  
    //       valor = valor.replace(/([0-9]{2})$/g, ",$1");


    //     if (valor.length > 6) {
    //         valor = valor.replace(/([0-9]{3}),([0-9]{2}$)/g, ".$1,$2");
    //     }
    //     this.venda.valorVenda = valor;
    //     if(valor == 'NaN') this.venda.valorVenda = '';

    //     // valor = valor.replace(/\./gi,'').replace(/,/gi,'.')
    // },
</script>

<style>

</style>