<template>
    <v-app>
    <v-app-bar app color="primary" dark>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-img
        max-height="30"
        max-width="30"
        src="../assets/logo.png"
        class="mr-3"
      ></v-img>
      <v-toolbar-title>Fish Farm</v-toolbar-title>
    </v-app-bar>


    <v-navigation-drawer v-model="drawer" dark app >
      <v-app-bar color="blue">
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title class="title">
              Faz. Canto do cedro
            </v-list-item-title>
            <v-list-item-subtitle>
              Luis
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-app-bar>

        <v-divider></v-divider>

        <v-list dense nav >
          <v-list-item v-for="item in items" :key="item.title"  :to="item.to">
            <v-list-item-icon>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-icon>

            <v-list-item-content>
              <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>  
            <!--  -->
      </v-navigation-drawer>

      <v-main>
        <v-container>
          <Nuxt />
        </v-container>
      </v-main>
      
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      clipped: false,
      drawer: true,
      fixed: false,
      items: [
          { title: 'Dashboard', icon: 'mdi-view-dashboard', to:'/' },
          { title: 'Ativos', icon: 'mdi-waves', to:'/ativos' },
          { title: 'Clientes', icon: 'mdi-account-heart', to: '/clientes'},
          { title: 'Vendas', icon: 'mdi-cash-plus', to:'/vendas' },
          { title: 'Despesas', icon: 'mdi-cash-minus', to:'/despesas' },
          { title: 'Categorias de Despesas', icon: 'mdi-tag-multiple', to: '/categorias'},
        ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js'
    }
  }
}
</script>
