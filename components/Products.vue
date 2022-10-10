<template>
  <v-dialog
    v-model="dialogProducts"
    fullscreen
    hide-overlay
    transition="dialog-bottom-transition"
  >
    <template v-slot:activator="{ on, attrs }">
      <v-btn color="primary" class="mr-3" dark v-bind="attrs" v-on="on">
        ดูสินค้า
      </v-btn>
    </template>
    <v-card>
      <v-toolbar dark color="primary">
        <v-btn icon dark @click="dialogProducts = false">
          <v-icon>mdi-close</v-icon>
        </v-btn>
        <v-spacer></v-spacer>
      </v-toolbar>
      <v-data-table
        :headers="headers"
        :items="listProduct"
        sort-by="name"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>สินค้า</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
                >
                  New Item
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.name"
                          label="ชื่อสินค้า"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.product"
                          label="ชื่อผลิตภัณฑ์"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.unitprice"
                          label="ราคาต่อหน่วย"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.unitofmeasure"
                          label="หน่วยวัด"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.productcategary"
                          label="หมวดหมู่สินค้า"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.productimage"
                          label="รูปภาพสินค้า"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.productdescription"
                          label="รายละเอียดสินค้า"
                        ></v-text-field>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">
                    Cancel
                  </v-btn>
                  <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="text-h5"
                  >Are you sure you want to delete this item?</v-card-title
                >
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete"
                    >Cancel</v-btn
                  >
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                    >OK</v-btn
                  >
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template>
      </v-data-table>
      <v-divider></v-divider>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  props: ["shop"],
  data: () => ({
    dialogProducts: false,
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "ชื่อสินค้า",
        align: "start",
        sortable: false,
        value: "name",
      },
      { text: "ชื่อผลิตภัณฑ์", value: "product" },
      { text: "ราคาต่อหน่วย", value: "unitprice" },
      { text: "หน่วยวัด", value: "unitofmeasure" },
      { text: "หมวดหมู่สินค้า", value: "productcategary" },
      { text: "รูปภาพสินค้า", value: "productimage" },
      { text: "รายละเอียดสินค้า", value: "productdescription" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    products: [],
    editedIndex: -1,
    editedItem: {
      name: "",
      product: "",
      unitprice: 0,
      unitofmeasure: 0,
      productcategary: "",
      productimage: [],
      productdescription: "",
    },
    defaultItem: {
      name: "",
      product: "",
      unitprice: 0,
      unitofmeasure: 0,
      productcategary: "",
      productimage: [],
      productdescription: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
    listProduct() {
      let product = [];
      if (this.products.length > 0) {
        this.products.forEach((element) => {
          if (element.shop == this.shop.id) {
            product.push(element);
          }
        });
      }
      return product;
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
    initialize() {
      this.products = [
        {
          name: "Frozen Yogurt",
          product: "",
          unitprice: 0,
          unitofmeasure: 0,
          productcategary: "",
          productimage: [],
          productdescription: "",
        },
      ];
    },

    editItem(item) {
      this.editedIndex = this.products.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.products.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.products.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.products[this.editedIndex], this.editedItem);
      } else {
        this.editedItem.shop = this.shop.id;
        this.products.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
