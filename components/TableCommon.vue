<template>
  <v-data-table :loading="load" loading-text="Carregando... Por favor aguarde" class="elevation-3" :headers="headers" :items="objs">
    
    <template v-slot:top>
        <v-toolbar flat >
            <v-toolbar-title>{{title}}</v-toolbar-title>
            <v-divider class="mx-4" inset vertical ></v-divider>
            

            <!-- <v-btn>Filtrar resultados</v-btn> -->

            <v-spacer></v-spacer>
            
            <CruidDialog :campos="camposDialog" :subCampos="subCampos" :title="title" :textoBtn="textoBtn" :dialogEdit="dialogEdit"                    
                  @saveObject="saveObject"
                  @dialogEdit="dialogEdit=$event"
                  @editObject="$emit('editObject', $event)"
                   />
                   <!--  -->

            <v-dialog v-model="dialogDelete" max-width="550px" persistent>
              <v-card>
                <v-card-title class="headline">Tem certeza que deseja excluir esta {{textoBtn}} ?</v-card-title>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="dialogDelete=!dialogDelete">Cancelar</v-btn>
                  <v-btn color="blue darken-1" text @click="deletObject">Confirmar</v-btn>
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
            
        </v-toolbar>
    </template>    

    <template v-slot:item="{item}" >
      <tr> 
        <td v-for="(keyObj, index) in Object.values(item)"  :key="index" :hidden="index==0" > 
            {{  writeItem(keyObj)  }}
            
              <!-- <v-avatar v-if="typeof keyObj === 'boolean'" 
                :color="keyObj ? 'green' : 'red'"
                size="26">
                <v-icon dark v-if="keyObj">mdi-checkbox-marked-circle</v-icon>
                <v-icon dark v-else>mdi-checkbox-blank-circle</v-icon>
              </v-avatar>   -->
              <v-simple-checkbox v-if="typeof keyObj === 'boolean'" 
                :value="keyObj"
                disabled
              ></v-simple-checkbox>



            <!-- {{ typeof keyObj === 'object' ? formatData(keyObj) : keyObj  }}  -->
            <!-- {{ {}.toString.call(keyObj).split(' ')[1].slice(0, -1).toLowerCase() === 'date' ? formatData(keyObj) : keyObj.name ? keyObj.name : keyObj }} -->
        </td>
        <td>
          <v-icon small class="mr-2" @click="loadEditObject(item)" >
            mdi-pencil
          </v-icon>
          <v-icon small @click="itemDelete = item; dialogDelete=true" >
            mdi-delete
          </v-icon>
        </td>
      </tr>      
    </template>


    <template v-slot:no-data>
        <p class="ma-4 title" >Não há {{title}} registradas</p>
      <!-- <v-btn color="primary" class="mb-3">
        Adicionar nova venda
      </v-btn> -->
    </template> 
    
  
  </v-data-table>
</template>

<script>
// import CruidDialog from '../common/CruidDialog'
import moment from 'moment'

export default {
    // components:{
    //     CruidDialog
    // },
    props:{
      load:Boolean,
      headers:Array,
      objs:Array,
      title:String,
      textoBtn: String,
      subObjs:Array
    },
    data: () => ({
        dialogDelete: false,
        dialogEdit: false,
        itemDelete:{},
        camposDialog:[],
        subCampos:[]
    }),
    created(){
      this.initialize()      
    },
    methods:{    
      initialize(){
        this.camposDialog = this.headers  
        // this.subCampos =[{
          
        // }]      
        //console.log(this.objs)
      },    
      saveObject(e){        
        this.$emit("saveObject", e)
      },
      loadEditObject(item){
        // const a = []
        // this.headers.map(obj => {
        //   obj.type === 'Object' 
        // })
        this.dialogEdit = item

      },
      // editObject(item){

      // },
      deletObject(){
        const indiceDelete = this.itemDelete.id
        this.$emit("deletObject", indiceDelete)
        this.dialogDelete=false
      },
      formatData(item){
        return moment(item).format('DD/MM/YYYY')
      },
      writeItem(item){
        const type = {}.toString.call(item).split(' ')[1].slice(0, -1).toLowerCase() 
        return type === 'date' ? this.formatData(item) : type ==='boolean' ? '' : item.name ? item.name : item        
      }
    }

}
</script>

<style>

</style>