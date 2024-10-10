<template>
  <div>
    <div v-if="isAdd" class="form">
      <v-card class="pa-5" width="400">
        <v-card-title class="justify-center">Thêm sản phẩm</v-card-title>
        <v-card-text>
          <v-form>
            <v-text-field
              v-model="addedProduct.name"
              label="Tên sản phẩm"
              :rules="rules"
              outlined
              required
            ></v-text-field>

            <v-text-field
              label="Hình ảnh (url)"
              :rules="rules"
              v-model="addedProduct.image"
              outlined
            ></v-text-field>

            <v-text-field
              label="Giá"
              :rules="rules"
              v-model="addedProduct.price"
              type="number"
              outlined
              required
            ></v-text-field>

            <v-text-field
              label="Số lượng"
              :rules="rulesQuantity"
              v-model="addedProduct.quantity"
              type="number"
              outlined
              required
            ></v-text-field>
          </v-form>
        </v-card-text>

        <v-card-actions class="justify-end">
          <v-btn @click="closeForm" color="grey" class="mr-4">Đóng</v-btn>
          <v-btn
            v-if="isEdit == false"
            @click="saveProduct"
            color="blue darken-1"
            dark
            >Lưu</v-btn
          >
          <v-btn
            v-if="isEdit == true"
            @click="editting"
            color="blue darken-1"
            dark
            >Lưu</v-btn
          >
        </v-card-actions>
      </v-card>
    </div>
    <div style="margin-bottom: 20px">
      <div>
        <v-btn color="primary" @click="openForm">Thêm mới sản phẩm</v-btn>
      </div>
    </div>
    <div>
      <v-table theme="dark">
        <thead>
          <tr>
            <th class="text-left"><b>#</b></th>
            <th class="text-left"><b>Tên sản phẩm</b></th>
            <th class="text-left"><b>Hình ảnh</b></th>
            <th class="text-left"><b>Giá</b></th>
            <th class="text-left"><b>Số lượng</b></th>
            <th class="text-left"><b>Ngày thêm</b></th>
            <th class="text-left"><b>Chức năng</b></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in products" :key="product.id">
            <td>{{ index }}</td>
            <td>{{ product.name }}</td>
            <td><img class="image" v-bind:src="product.image" alt="" /></td>
            <td>{{ product.price }}</td>
            <td>{{ product.quantity }}</td>
            <td>{{ product.date }}</td>
            <td class="btn">
              <v-btn
                variant="tonal"
                @click="removeProduct(product.id)"
                class="del"
                >Delete</v-btn
              >
              <v-btn variant="tonal" @click="edit(product)" color="warning"
                >Edit</v-btn
              >
            </td>
          </tr>
        </tbody>
      </v-table>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from "vue";

/*
API
1. Tổng quan API
2. HTTP
3. JSON Mock API
4. Fetch API vs Vue.js
*/

//Khai báo hàm lấy dữ liệu

const rules = reactive([(value) => !!value || "Required."]);
const rulesQuantity = reactive([
  (value) => !!value || "Required.",
  (value) => value > 0 || "Quantity must greater than 0",
]);

let addedProduct = reactive({
  name: "",
  image: "",
  price: null,
  quantity: null,
  date: new Date().toISOString().split("T")[0],
});

const saveProduct = async () => {
  const newProduct = addedProduct;
  await fetch("http://localhost:3000/products", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(newProduct),
  });
  addedProduct = {
    name: "",
    image: "",
    price: null,
    quantity: null,
    date: new Date().toISOString().split("T")[0],
  };
  getData();
  isAdd.value = true;
};
const isAdd = ref(false);
const isEdit = ref(false);
const edittingUSer = ref(null);
const input = ref("");
const products = ref([]);
const getData = async () => {
  const response = await fetch("http://localhost:3000/products");
  const data = await response.json();
  products.value = data;
};

const closeForm = () => {
  isAdd.value = false;
  isEdit.value = false;
  addedProduct = {
    name: "",
    image: "",
    price: null,
    quantity: null,
    date: new Date().toISOString().split("T")[0],
  };
};

const openForm = () => {
  isAdd.value = true;
};

const removeProduct = async (id) => {
  const confirm = window.confirm("Are you sure you want to delete");
  if (confirm) {
    console.log(id);

    await fetch(`http://localhost:3000/products/${id}`, {
      method: "DELETE",
    });
    getData();
  }
};

const edit = async (product) => {
  openForm();
  isEdit.value = true;
  edittingUSer.value = product;
  console.log(edittingUSer.value, 999);
  // addedProduct = edittingUSer.value;
};

const editting = async () => {
  const newName = input.value;
  await fetch(`http://localhost:3000/products/${edittingUSer.value.id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ name: newName }),
  });
  getData();
  input.value = "";
  isEdit.value = false;
};

const cancel = () => {
  isEdit.value = false;
  input.value = "";
};

onMounted(() => {
  getData();
});
</script>

<style>
.del {
  background-color: red;
  margin-right: 10px;
}
.image {
  width: 100px !important;
}
.form {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
</style>
