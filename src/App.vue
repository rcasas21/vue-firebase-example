<template>
  <div id="app">
    <nav class="navbar">
      <el-button v-on:click="loggin" v-if="user && !user.uid"
        >Iniciar sesión</el-button
      >
      <el-button type="danger" v-on:click="logOut" v-if="user && user.uid"
        >Cerrar sesión</el-button
      >

      <el-button v-on:click="getStores" v-if="stores.length === 0"
        >Listar tiendas</el-button
      >
      <el-button type="warning" v-on:click="hideStores" v-if="stores.length > 0"
        >Ocultar tiendas</el-button
      >
    </nav>
    <div class="content">
      <div class="logged" v-if="user && user.uid">
        Welcome <span>{{ user && user.uid }}</span>
      </div>
      <div v-if="stores.length > 0">
        Add Item
        <input v-model="storeAddItem" placeholder="Select Store Id"/>
        <input v-model="itemToAdd" placeholder="Add Item"/>
        <el-button class="el-icon-circle-plus" @click="addItem()"></el-button>
      </div>
      <div class="stores" v-if="stores.length > 0">
        <el-card class="store" v-for="(store, i) in stores" :key="i">
          <div class="nombre">
            Id: <span>{{ store.id }}</span>
          </div>
          <div class="nombre">
            Nombre: <span>{{ store.nombre }}</span>
          </div>
          <div class="telefono">
            Telefono: <span>{{ store.telefono }}</span>
          </div>
          <div class="pedidos">
            <div>Pedidos:
            </div>
            <ul v-if="store.pedidos.length > 0">
              <li class="pedido" v-for="(pedido, i) in store.pedidos" :key="i">
                <div>
                  <span>{{ pedido }}</span>
                  <i class="el-icon-delete" @click="deleteItem(store, pedido)"></i>
                </div>
              </li>
            </ul>
            <div v-else class="empty">No hay pedidos por el momento</div>
          </div>
          <el-button
            class="delete-button"
            type="danger"
            icon="el-icon-close"
            circle
            v-on:click="deleteStore(store.id)"
          />
        </el-card>
        <div class="add-button-wrapper">
          <el-button
            type="success"
            icon="el-icon-plus"
            circle
            v-on:click="toggleDialog"
          />
        </div>
      </div>
    </div>
    <el-dialog title="Nueva Tienda" :visible.sync="dialogFormVisible">
      <el-form label-position="top" :model="form">
        <el-form-item label="id Tienda" :label-width="formLabelWidth">
          <el-input-number v-model="form.id" :min="1"></el-input-number>
        </el-form-item>
        <el-form-item label="Nombre Tienda" :label-width="formLabelWidth">
          <el-input v-model="form.nombre"></el-input>
        </el-form-item>
        <el-form-item label="Teléfono Tienda" :label-width="formLabelWidth">
          <el-input v-model="form.telefono"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">Cancel</el-button>
        <el-button type="primary" @click="addStore">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// import axios from 'axios'
import { firebaseAuth, db, fieldValue } from "../firebase.js";

export default {
  name: "App",
  data() {
    return {
      dialogFormVisible: false,
      form: this.setForm(),
      formLabelWidth: "120px",
      user: {},
      stores: [],
      storeAddItem: '',
      itemToAdd: ''
    };
  },
  methods: {
    loggin() {
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
        }
      });
    },
    logOut() {
      let that = this;
      const user = firebaseAuth.currentUser;
      user
        .delete()
        .then(function() {
          // User deleted.
          that.user = {};
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    getStores() {
      let that = this;
      db.collection("Tiendas").onSnapshot((querySnapshot) => {
        querySnapshot.docChanges().forEach((change) => {
          let modifiedIndex = [];
          switch (change.type) {
            case "added":
              that.stores.push(change.doc.data());
              console.log("added");
              break;
            case "modified":
              modifiedIndex = that.stores.findIndex(
                (item) => item.id === change.doc.data().id
              );
              that.stores[modifiedIndex].nombre = change.doc.data().nombre;
              that.stores[modifiedIndex].telefono = change.doc.data().telefono;
              that.stores[modifiedIndex].pedidos = change.doc.data().pedidos;
              console.log("modified");
              return;
            case "removed":
              console.log("Removed from BBDD");
              that.stores = that.stores.filter(
                (item) => item.id !== change.doc.data().id
              );
              break;
          }
        });
      });
    },
    hideStores() {
      this.stores = [];
    },
    addStore() {
      let that = this;
      db.collection("Tiendas")
        .doc(that.form.id.toString())
        .set(that.form)
        .then(() => {
          that.toggleDialog();
          that.form = that.setForm();
        });
    },
    setForm() {
      return {
        id: "",
        nombre: "",
        telefono: "",
        pedidos: [],
      };
    },
    deleteStore(id) {
      db.collection("Tiendas")
        .doc(id.toString())
        .delete()
        .then(function() {
          console.log("Document successfully deleted!");
        })
        .catch(function(error) {
          console.error("Error removing document: ", error);
        });
    },
    addItem() {
      console.log(this.storeAddItem)
      let datStore = this.storeAddItem
      let datItem = this.itemToAdd
      db.collection("Tiendas")
              .doc(datStore)
              .update({
                "pedidos": fieldValue.arrayUnion(datItem)
              })
              .then(function() {
                console.log("Document successfully updated!");
              })
              .catch(function(error) {
                console.error("Error removing document: ", error);
              });
    },
    deleteItem(store, item) {
      db.collection("Tiendas")
          .doc(store.id.toString())
          .update({
                "pedidos": fieldValue.arrayRemove(item)
          })
          .then(function() {
            console.log("Document successfully updated!");
          })
          .catch(function(error) {
            console.error("Error removing document: ", error);
          });
    toggleDialog() {
      this.dialogFormVisible = !this.dialogFormVisible;
    },
  },
};
</script>

<style type="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.navbar {
}

.content {
  padding: 24px;
}

.content .logged {
  margin-bottom: 24px;
  font-size: 20px;
}

.content .logged span {
  font-weight: bold;
}

.content .stores {
  display: flex;
  flex-wrap: wrap;
  position: relative;
}

.content .stores .store {
  margin: 8px;
  width: 30%;
  text-align: left;
  position: relative;
}

.store span {
  font-size: 13px;
  font-style: italic;
}

.delete-button {
  position: absolute;
  top: -8px;
  right: -8px;
}

ul {
  margin: 0;
}

.empty {
  font-style: italic;
  font-weight: bold;
  font-size: 12px;
}

.add-button-wrapper {
  display: flex;
  justify-content: flex-end;
  align-items: flex-end;
  position: absolute;
  right: 0;
  top: 0;
}
</style>
