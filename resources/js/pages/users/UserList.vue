<script setup>
import { onMounted, reactive, ref, watch } from "vue";
import axios from "axios";
import { Form, Field } from "vee-validate";
import * as yup from "yup";
import { debounce } from "lodash";
import { Bootstrap4Pagination } from "laravel-vue-pagination";
import { useToaster } from "../../toaster.js";
import UserListItem from "./UserListItem.vue";

const toaster = useToaster();
const users = ref({ data: [] });
const editing = ref(false);
const formValues = ref();
const form = ref(null);
const searchQuery = ref(null);
const selectedUsers = ref([]);

const getUsers = (page = 1) => {
  axios
    .get(`/api/users?page=${page}`, {
      params: {
        query: searchQuery.value,
      },
    })
    .then((res) => {
      users.value = res.data;
      selectedUsers.value = [];
      selectAll.value = false;
    })
    .catch((err) => {
      console.log(err);
    });
};

const createUser = (values, { resetForm, setErrors }) => {
  axios
    .post("/api/users", values)
    .then((res) => {
      users.value.data.unshift(res.data);
      $("#userFormModal").modal("hide");
      resetForm();
      toaster.success("User created successfully");
    })
    .catch((err) => {
      if (err.response.data.errors) {
        setErrors(err.response.data.errors);
      }
    });
};

const createUserSchema = yup.object({
  name: yup.string().required(),
  email: yup.string().email().required(),
  password: yup.string().required().min(8),
});

const editUserSchema = yup.object({
  name: yup.string().required(),
  email: yup.string().email().required(),
  password: yup.string().when((password, schema) => {
    return password ? schema.required().min(8) : schema;
  }),
});

const addUser = () => {
  editing.value = false;
  $("#userFormModal").modal("show");
};

const editUser = (user) => {
  editing.value = true;
  form.value.resetForm();
  formValues.value = {
    id: user.id,
    name: user.name,
    email: user.email,
  };
  $("#userFormModal").modal("show");
};

const updateUser = (values, { setErrors }) => {
  axios
    .put(`/api/users/${formValues.value.id}`, values)
    .then((res) => {
      const index = users.value.findIndex((user) => user.id === res.data.id);
      users.value[index] = res.data;
      $("~userFormModal").modal("hide");
      toaster.success("User updated successfully");
    })
    .catch((err) => {
      setErrors(err.response.data.errors);
    })
    .finally(() => {
      form.value.resetForm();
    });
};

const handleSubmit = (values, actions) => {
  console.log(actions);
  if (editing.value) {
    updateUser(values, actions);
  } else {
    createUser(values, actions);
  }
};

const toggleSelection = (user) => {
  const index = selectedUsers.value.indexOf(user.id);
  if (index === -1) {
    // means no index was found
    selectedUsers.value.push(user.id);
  } else {
    selectedUsers.value.splice(index, 1);
  }
};

const userIdBeingDeleted = ref(null);

const confirmUserDeletion = (userId) => {
  userIdBeingDeleted.value = userId;
  $("#deleteUserModal").modal("show");
};

const deleteUser = () => {
  axios.delete(`/api/users/${userIdBeingDeleted.value}`).then((res) => {
    $("#deleteUserModal").modal("hide");

    toaster.success("User deleted successfully");
    users.value.data = users.value.data.filter(
      (user) => user.id !== userIdBeingDeleted.value
    );
  });
};

const bulkDelete = () => {
  axios
    .delete(`/api/users`, {
      data: {
        ids: selectedUsers.value,
      },
    })
    .then((res) => {
      users.value.data = users.value.data.filter(
        (user) => !selectedUsers.value.includes(user.id)
      );
      selectedUsers.value = [];
      selectAll.value = false;
      toaster.success(res.data.message);
    });
};

const selectAll = ref(false);
const selectAllUsers = () => {
  if (selectAll.value) {
    selectedUsers.value = users.value.data.map((user) => user.id);
  } else {
    selectedUsers.value = [];
  }
};

watch(
  searchQuery,
  debounce(() => {
    getUsers();
  }, 800)
);

onMounted(() => {
  getUsers();
});
</script>


<template>
  <div class="content-header">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-6">
          <h1 class="m-0">Users</h1>
        </div>
        <div class="col-sm-6">
          <ol class="breadcrumb float-sm-right">
            <li class="breadcrumb-item"><a href="#">Home</a></li>
            <li class="breadcrumb-item active">Users</li>
          </ol>
        </div>
      </div>
    </div>
  </div>

  <div class="content">
    <div class="d-flex justify-content-between">
      <div>
        <button type="button" class="mb-2 btn btn-primary" @click="addUser">
          <i class="fa fa-plus-circle mr-1"></i>
          Add New User
        </button>
        <div class="d-flex" v-if="selectedUsers.length > 0">
          <button
            type="button"
            class="mb-2 btn btn-danger ml-2"
            @click="bulkDelete"
          >
            <i class="fa fa-trash mr-1"></i>
            Delete Selected
          </button>
          <span class="ml-2">Selected {{ selectedUsers.length }} users</span>
        </div>
      </div>
      <div>
        <input
          type="text"
          name=""
          v-model="searchQuery"
          class="form-control"
          placeholder="search"
        />
      </div>
    </div>
    <div class="container-fluid">
      <div class="card">
        <div class="card-body">
          <table class="table table-bordered">
            <thead>
              <tr>
                <th>
                  <input
                    type="checkbox"
                    name=""
                    @change="selectAllUsers"
                    v-model="selectAll"
                    id=""
                  />
                </th>
                <th style="width: 10px">#</th>
                <th>Name</th>
                <th>Email</th>
                <th>Registered Date</th>
                <th>Role</th>
                <th>Options</th>
              </tr>
            </thead>
            <tbody v-if="users.data.length > 0">
              <UserListItem
                v-for="(user, index) in users.data"
                :key="user.id"
                :user="user"
                :index="index"
                @edit-user="editUser"
                @toggle-selection="toggleSelection"
                :select-all="selectAll"
                @confirm-user-deletion="confirmUserDeletion"
              />
            </tbody>
            <tbody v-else>
              <tr>
                <td colspan="6" class="text-center">No results found</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <Bootstrap4Pagination :data="users" @pagination-change-page="getUsers" />
    </div>
  </div>

  <!-- Modal -->
  <div
    class="modal fade"
    id="userFormModal"
    data-backdrop="static"
    tabindex="-1"
    role="dialog"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">
            <span v-if="editing">Edit User</span>
            <span v-else>Add New User</span>
          </h5>
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <Form
          ref="form"
          @submit="handleSubmit"
          :validation-schema="editing ? editUserSchema : createUserSchema"
          :initial-values="formValues"
          v-slot="{ errors }"
        >
          <div class="modal-body">
            <div class="form-group">
              <label for="name">Name</label>
              <Field
                type="text"
                class="form-control"
                name="name"
                id="name"
                aria-describedby="nameHelp"
                placeholder="Enter full name"
                :class="{ 'is-invalid': errors.name }"
              />
              <span class="invalid-feedback">{{ errors.name }}</span>
            </div>

            <div class="form-group">
              <label for="email">Email</label>
              <Field
                type="email"
                class="form-control"
                name="email"
                id="email"
                aria-describedby="nameHelp"
                placeholder="Enter email"
                :class="{ 'is-invalid': errors.email }"
              />
              <span class="invalid-feedback">{{ errors.email }}</span>
            </div>

            <div class="form-group">
              <label for="email">Password</label>
              <Field
                type="password"
                class="form-control"
                name="password"
                id="password"
                aria-describedby="nameHelp"
                placeholder="Enter password"
                :class="{ 'is-invalid': errors.password }"
              />
              <span class="invalid-feedback">{{ errors.password }}</span>
            </div>
          </div>

          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-dismiss="modal"
            >
              Cancel
            </button>
            <button type="submit" class="btn btn-primary">Save</button>
          </div>
        </Form>
      </div>
    </div>
  </div>

  <div
    class="modal fade"
    id="deleteUserModal"
    data-backdrop="static"
    tabindex="-1"
    role="dialog"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">
            <span>Delete User</span>
          </h5>
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <h5>Are you sure you want to delete this user</h5>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal">
            Cancel
          </button>
          <button
            type="button"
            @click.prevent="deleteUser"
            class="btn btn-primary"
          >
            Delete User
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
