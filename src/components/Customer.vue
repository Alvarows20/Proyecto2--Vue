<template>
  <v-layout align-start>
    <v-flex>
      <v-data-table
        :headers="headers"
        :items="customers"
        :search="search"
        sort-by="calories"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>Clientes</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-text-field class="text-xs-center" v-model="search"  label="Busqueda" :append-icon="icons.mdiMagnify " single-line  hide-details></v-text-field>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="700px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
                >
                  Nuevo Cliente
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-form
                      ref="form"
                      v-model="valid_form"
                      
                    >
                      <v-row>
                        <v-col cols="12" sm="12" md="6" class="p-0">
                          <v-text-field
                            v-model="editedItem.first_name"
                            label="Nombres Cliente"
                            :rules="[rules.required]"
                            autocomplete="disabled"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="12" md="6" class="p-0">
                          <v-text-field
                            v-model="editedItem.last_name"
                            label="Apellidos Cliente"
                            :rules="[rules.required]"
                            autocomplete="nope"
                          ></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="12" md="6" class="p-0">
                          <v-text-field
                            v-model="editedItem.email"
                            label="Email"
                            :rules="[rules.required, rules.email]"
                            autocomplete="new"
                          ></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="12" md="6" class="p-0">
                          <v-text-field
                            v-model="editedItem.nit"
                            label="NIT"
                            autocomplete="nope"
                          ></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="12" md="12" class="p-0">
                          <v-text-field
                            v-model="editedItem.address"
                            label="Direccion"
                            :rules="[rules.required]"
                            autocomplete="nope"
                          ></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="12" md="6" class="p-0">
                          <v-text-field
                            v-model="editedItem.phone"
                            label="Telefono"
                            autocomplete="nope"
                          ></v-text-field>
                        </v-col>

                        <v-col cols="12" sm="12" md="6" class="p-0">
                        <v-checkbox
                          v-model="editedItem.is_active"
                          label="Activo"
                        ></v-checkbox>
                        </v-col>
                      </v-row>
                    </v-form>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">
                    Cancelar
                  </v-btn>
                  <v-btn color="blue darken-1" text @click="save"> Guardar </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="text-h5"
                  >Estas seguro de eliminar este cliente</v-card-title
                >
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete"
                    >Cancelar</v-btn
                  >
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                    >Aceptar</v-btn
                  >
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data>
          <v-btn  @click="initialize"> No hay datos disponibles </v-btn>
        </template>
      </v-data-table>
    </v-flex>
    <v-overlay :value="overlay">
        <v-progress-circular
          indeterminate
          size="64"
        ></v-progress-circular>
      </v-overlay>
  </v-layout>
</template>
<script>

import axios from 'axios'
import {
  mdiMagnify 
} from '@mdi/js'

export default {
  data() {
    return {
      rules: {
        required: value => !!value || 'Campo obligatorio',
        email: value => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
          return pattern.test(value) || 'Email no valido'
        },
      },
      valid_form: true,
      search:'',
      overlay:false,
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: "Nombres Cliente",
          align: "left",
          sortable: true,
          value: "first_name",
        },
        {
          text: "Apellidos Cliente",
          align: "left",
          sortable: true,
          value: "last_name",
        },
        {
          text: "Email",
          align: "left",
          sortable: true,
          value: "email",
        },
        {
          text: "NIT",
          align: "left",
          sortable: true,
          value: "nit",
        },
        {
          text: "Direccion",
          align: "left",
          sortable: true,
          value: "address",
        },
        {
          text: "Telefono",
          align: "left",
          sortable: true,
          value: "phone",
        },
        { text: "Estado", value: "text_active" },
        { text: 'Acciones', value: 'actions', sortable: false },
      ],
      customers: [],
      editedIndex: -1,
      editedItem: {
        email: "",
        first_name:"",
        last_name:"",
        nit:"",
        address:"",
        phone:"",
        is_active: 1,
      },
      defaultItem: {
        email: "",
        first_name:"",
        last_name:"",
        nit:"",
        address:"",
        phone:"",
        is_active: 1,
      },
      icons:{
        mdiMagnify ,
      }
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Agregar Cliente" : "Editar Cliente";
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },
  created() {
    this.initialize();
  },
  methods: {
    get_status_text(status){
      if(status == true || status  ==1){
        return 'Activo'
      }else{
        return 'Inactivo'
      }
    },
    initialize() {
        let $this = this;
        this.overlay = true;
        axios.get("customer").then((response)=>{
            $this.customers = response.data
            $this.customers.forEach(function(item){
              item.text_active = $this.get_status_text(item.is_active)
            })
            $this.overlay = false;
        }).catch(()=>{
          this.overlay = false;
          this.$swal("Error", "Ocurrio un error al realizar la operacion :(", "error");
        })
    },

    editItem(item) {
      this.editedIndex = this.customers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.customers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.overlay = true;
      let $this = this;
      let deleted_item = this.customers[this.editedIndex]
      let edited_index = this.editedIndex

      axios.delete("customer/"+deleted_item.customer_id).then(()=>{
          $this.customers.splice(edited_index, 1);
          console.log(JSON.stringify($this.customers[edited_index]))
          $this.overlay = false;
      }).catch(()=>{
        this.overlay = false;
        this.$swal("Error", "Ocurrio un error al realizar la operacion :(", "error");
      })
      this.closeDelete();
      
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
      this.$refs.form.resetValidation();
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
      if(this.$refs.form){
        this.$refs.form.resetValidation();
      }
    },

    save() {
      this.$refs.form.validate();
      
      if(!this.valid_form){
        return;
      }

      this.overlay = true;
      if (this.editedIndex > -1) {
        let $this = this;

        axios.put("customer/"+this.editedItem.customer_id ,this.editedItem).then(()=>{
            $this.initialize();
        }).catch(()=>{
          this.overlay = false;
          this.$swal("Error", "Ocurrio un error al realizar la operacion :(", "error");
        })

      } else {
        let $this = this;
        axios.post("customer",this.editedItem).then((response)=>{
            let data = response.data
            data.text_active = $this.get_status_text(data.is_active)
            $this.customers.push(data);
            $this.overlay = false;
        }).catch(()=>{
          this.overlay = false;
          this.$swal("Error", "Ocurrio un error al realizar la operacion :(", "error");
        })
        
      }
      this.close();
      this.$refs.form.resetValidation()
    },
  },
};
</script>



<style>
.p-0{
  padding:0px 15px 0px 15px !important;
}
.swal2-title, .swal2-html-container, .swal2-confirm {
  font-family: Arial, Helvetica, sans-serif !important;
}
</style>