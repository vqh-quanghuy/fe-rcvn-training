<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="products.data"
      disable-sort
      class="elevation-2"
      hide-default-footer
      disable-pagination
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>List of Products</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="600px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Add Product
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-form ref="editedForm">
                    <v-row>
                      <v-col cols="12" sm="6" md="6">
                        <v-text-field
                          v-model="editedItem.product_name"
                          label="Name"
                          validate-on-blur
                          :rules="productNameRules"
                          :error-messages="editedItemErrors.product_name"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="6">
                        <v-text-field
                          v-model="editedItem.product_price"
                          type="number"
                          min="0"
                          label="Price"
                          validate-on-blur
                          :rules="productPriceRules"
                          :error-messages="editedItemErrors.product_price"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="12" md="12">
                        <v-file-input
                          :rules="productImageRules"
                          accept="image/png, image/jpeg, image/jpg"
                          prepend-icon="mdi-camera"
                          label="Product Image"
                          :error-messages="editedItemErrors.product_image"
                        ></v-file-input>
                      </v-col>
                      <v-col cols="12" sm="12" md="12">
                        <v-textarea
                          v-model="editedItem.description"
                          label="Description"
                          validate-on-blur
                          :error-messages="editedItemErrors.description"
                        ></v-textarea>
                      </v-col>
                      <v-col cols="12" sm="6" md="6">
                        <v-checkbox
                          v-model="editedItem.is_sale"
                          label="Is Sale"
                          :error-messages="editedItemErrors.is_sale"
                        ></v-checkbox>
                      </v-col>
                    </v-row>
                  </v-form>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="secondary" text @click="close">Cancel</v-btn>
                <v-btn color="primary" text @click="save">Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="600px">
            <v-card>
              <v-card-title class="text-h5"
                >Are you sure you want to delete this product?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="secondary" text @click="closeDelete"
                  >Cancel</v-btn
                >
                <v-btn color="primary" text @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon medium class="mx-1" color="accent" @click="editItem(item)"
          >mdi-pencil</v-icon
        >
        <v-icon medium color="error" @click="deleteItem(item)"
          >mdi-delete</v-icon
        >
      </template>
      <template v-slot:[`item.product_id`]="{ item }">
        <v-menu open-on-hover top offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              text
              v-bind="attrs"
              v-on="on"
            >
              {{item.product_id}}
            </v-btn>
          </template>
          <v-img
            lazy-src="https://picsum.photos/id/11/10/6"
            max-height="150"
            max-width="250"
            :src="item.product_image"
          ></v-img>
        </v-menu>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="load"> Reset </v-btn>
      </template>
      <template v-slot:[`item.index`]="{ index }">
        {{ index + 1 }}
      </template>
      <template v-slot:[`item.is_sale`]="{ item }">
        <v-chip v-if="item.is_sale === 1" color="green">
          <b>Sale</b>
        </v-chip>
        <v-chip v-else color="red">
          <b>Out of stock</b>
        </v-chip>
      </template>
    </v-data-table>
    <v-row class="text-center px-4 align-center" wrap>
      <v-col class="text-truncate" cols="12" md="2">
        Total {{ products.total }} records
      </v-col>
      <v-col cols="12" md="5">
        <v-pagination v-model="page" :length="products.last_page">
        </v-pagination>
      </v-col>
      <v-col cols="6" md="3">
        <v-select
          dense
          outlined
          hide-details
          :value="itemsPerPage"
          label="Items per page"
          @change="itemsPerPage = parseInt($event, 10)"
          :items="perPageChoices"
        >
        </v-select>
      </v-col>
      <v-col cols="6" md="2">
        <v-text-field
          v-model="page"
          label="Go to page"
          type="number"
          outlined
          hide-details
          dense
          min="1"
          :max="products.last_page"
          @input="
            page = parseInt($event, 10);
            page =
              page > products.last_page
                ? products.last_page
                : page < 1
                ? 1
                : page;
          "
        ></v-text-field>
      </v-col>
    </v-row>
  </div>
</template>
<script>
export default {
  name: "ProductList",
  data() {
    return {
      products: {},
      dialog: false,
      dialogDelete: false,
      dialogDeactivate: false,
      headers: [
        { text: "#", value: "index" },
        { text: "Product Code", align: "start", value: "product_id" },
        {
          text: "Product Name",
          align: "start",
          value: "product_name",
          width: "20%",
        },
        { text: "Price", value: "product_price" },
        { text: "Description", value: "description", width: "20%" },
        { text: "Status", value: "is_sale", align: "center" },
        { text: "Actions", value: "actions", align: "center" },
      ],
      page: 1,
      itemsPerPage: 10,
      perPageChoices: [
        { text: "5 records/page", value: 5 },
        { text: "10 records/page", value: 10 },
        { text: "20 records/page", value: 20 },
      ],
      editedIndex: -1,
      editedItem: {
        product_name: "",
        product_price: "",
        product_image: "",
        description: "",
        product_id: "",
        is_sale: true, //True: 1 => Sale,  False: 0 => Out of stock
      },
      defaultItem: {
        product_name: "",
        product_price: "",
        product_image: "",
        description: "",
        product_id: "",
        is_sale: true, //True: 1 => Sale,  False: 0 => Out of stock
      },
      editedItemErrors: {
        product_name: [],
        product_price: [],
        product_image: [],
        description: [],
        is_sale: [],
      },

      // Validation rule
      productNameRules: [
        (v) => !!v || "Product name is required.",
        (v) =>
          (v && v.length <= 255) ||
          "Product name must be less than 255 characters.",
      ],
      productPriceRules: [
        (v) => !!v || "Product price is required.",
        (v) => /^\d*\.?\d*$/.test(v) || "Price must be valid.",
      ],
      productImageRules: [
        (v) =>
          !v ||
          v.size < 2000000 ||
          "Product image size should be less than 2 MB.",
      ],

      apiHeaders: {
        "Content-Type": "application/json",
        Accept: "application/json",
        Authorization: "Bearer " + sessionStorage.getItem("access_token"),
      },
    };
  },
  methods: {
    async load(page, itemsPerPage) {
      await this.$axios
        .get(`${this.$backendUrl}user/products/`, {
          headers: this.apiHeaders,
          params: {
            page: page,
            per_page: itemsPerPage,
          },
        })
        .then((res) => {
          if (res.status === 200) {
            const DATA = res.data.data;
            this.products = DATA;
          }
        })
        .catch((err) => {
          if (err.status !== 200) console.error(err.response.data.message);
        });
    },
    editItem(item) {
      this.editedIndex = 1;
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.itemIdToDelete = item.id;
      this.dialogDelete = true;
    },

    async deleteItemConfirm() {
      await this.$axios
        .delete(`${this.$backendUrl}user/products/${this.itemIdToDelete}`, {
          headers: this.apiHeaders,
        })
        .then((res) => {
          alert(res.data.message);
          this.load(this.page, this.itemsPerPage);
        })
        .catch((err) => {
          alert(err.response.data.message);
        });

      this.closeDelete();
    },

    close() {
      this.editedItemErrors = {};
      this.$refs.editedForm.reset();
      this.dialog = false;
    },

    closeDelete() {
      this.dialogDelete = false;
      this.itemIdToDelete = "";
    },

    debug(msg) {
      console.log(msg);
    },

    async save() {
      if (!this.$refs.editedForm.validate()) return;

      let editedItem = this.editedItem;
      if (this.editedIndex > -1) {
        // Call to Edit axios
        await this.$axios
          .put(
            `${this.$backendUrl}user/products/${editedItem.id}`,
            {
              product_name: editedItem.product_name,
              product_price: editedItem.product_price,
              product_image: editedItem.product_image,
              description: editedItem.description,
              is_sale: editedItem.is_sale ? 1 : 0,
            },
            {
              headers: this.apiHeaders,
            }
          )
          .then((res) => {
            alert(res.data.message);
            this.load(this.page, this.itemsPerPage);
            this.close();
          })
          .catch((err) => {
            this.editedItemErrors = err.response.data.error;
            alert(err.response.data.message);
          });
      } else {
        // Call to Create axios
        await this.$axios
          .post(
            `${this.$backendUrl}user/products`,
            {
              product_name: editedItem.product_name,
              product_price: editedItem.product_price,
              product_image: editedItem.product_image,
              description: editedItem.description,
              is_sale: editedItem.is_active ? 1 : 0,
            },
            {
              headers: this.apiHeaders,
            }
          )
          .then((res) => {
            alert(res.data.message);
            this.load(this.page, this.itemsPerPage);
            this.close();
          })
          .catch((err) => {
            this.editedItemErrors = err.response.data.error;
            alert(err.response.data.message);
          });
      }
    },
    roleName(roleId) {
      return this.roles.find((x) => x.id === parseInt(roleId)).role_name;
    },
    clearEmailErrors() {
      this.editedItemErrors.email = [];
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
    page(val) {
      this.load(val, this.itemsPerPage);
    },
    itemsPerPage(val) {
      this.load(this.page, val);
    },
  },
  mounted() {
    this.load(this.page, this.itemsPerPage);
  },
};
</script>