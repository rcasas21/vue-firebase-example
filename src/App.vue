<template>
  <div id="app">
    <div class="logged" v-if="user.uid">Login: {{user.uid}}</div>
    <div class="tienda" v-for="(tienda, i) in tiendas" :key="i">
      <div class="nombre">Nombre: {{tienda.nombre}}</div>
      <div class="telefono">Telefono: {{tienda.telefono}}</div>
      <div class="pedidos">
        <div v-if="tienda.pedidos.length > 0">Pedidos:</div>
        <div class="pedido" v-for="(pedido, i) in tienda.pedidos" :key="i">
          <div>{{pedido}}</div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
// import axios from 'axios'
import { firebaseAuth, db } from '../firebase.js'

export default {
  name: 'App',
  data() {
    return {
      user : {},
      tiendas: []
    }
  },
  methods: {
    init () {
      let that = this;
      db.collection('Tiendas')
          // .where("timestamp", ">=", that.date)
          // .where(
          //  "timestamp",
          //  "<=",
           // that.config.netxDays[that.config.netxDays.length - 1]
          //)
          .onSnapshot((querySnapshot) => {
            querySnapshot.docChanges().forEach((change) => {
              let modifiedIndex = [];
              switch (change.type) {
                case "added":
                  that.tiendas.push(change.doc.data());
                  console.log('added');
                  break;
                case "modified":
                  modifiedIndex = that.tiendas.findIndex(
                    (item) => item.id === change.doc.data().id
                  );
                  that.tiendas[modifiedIndex].nombre = change.doc.data().nombre;
                  that.tiendas[modifiedIndex].telefono = change.doc.data().telefono;
                  that.tiendas[modifiedIndex].pedidos = change.doc.data().pedidos;
                  console.log('modified');
                  return;
                case "removed":
                  console.log("Removed from BBDD");
                  break;
              }
            });
          });
      
    }
  },
  created () {
    let that = this;
    firebaseAuth.signInAnonymously().catch(function(error) {
      // Handle Errors here.
      var errorCode = error.code;
      var errorMessage = error.message;
      console.log(errorCode, errorMessage);
      // ...
    });
    firebaseAuth.onAuthStateChanged(function(user) {
      if (user) {
        that.user = user;
        that.init();
      }
    });
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.logged {
  margin-bottom: 20px;
}

.tienda {
  border-radius: 4px;
  border: 1px solid grey;
  padding: 16px;
  margin-bottom: 8px;
}
</style>
