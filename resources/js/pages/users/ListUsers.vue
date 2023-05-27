<script setup>
import axios from 'axios';
import { ref, onMounted, reactive} from 'vue';
import {Form, Field } from 'vee-validate';
import * as yup from 'yup';


const users = ref([]);
const editing = ref(false);
const formValues = ref();
const form = ref(null);

const getUsers = () => {
  axios.get('/api/users')
  .then((response) => {
    users.value = response.data;
  })
};

const createUserSchema = yup.object({
  name: yup.string().required(),
  email: yup.string().email().required(),
  password: yup.string().required().min(8),
});

const editUserSchema = yup.object({
  name: yup.string().required(),
  email: yup.string().email().required(),
  // password: yup.string().when((password, schema) => {
  //   //TODO: fix, password check to 'undefined'. This always true
  //   return password ? schema.required().min(8) : schema;
  // }),
  password: yup.string().notRequired().test('password', 'Passwords must be be minimum of 8 characters', function(value) {
    if (!!value) {
      const schema = yup.string().min(8);
      return schema.isValidSync(value);
    }
    return true;
  }),
})

const createUser = (values, {resetForm, setFieldError }) => {
  axios.post('/api/users', values)
      .then((response) => {
        users.value.data.unshift(response.data);
        $('#userFormModal').modal('hide');
        useResetForm();
      })
  .catch((error) => {
    setFieldError('email', error.response.data.errors.email[0]);
  })
};

const addUser = () => {
  editing.value = false;
  $('#userFormModal').modal('show');
};

const editUser = (user) => {
  form.value.resetForm();
  editing.value = true;
  $('#userFormModal').modal('show');
  formValues.value = {
    id: user.id,
    name: user.name,
    email: user.email
  };
};

const updateUser = (values) => {
  console.log(formValues);
  console.log(values);
  axios.put('/api/users/' + formValues.value.id, values)
  .then((response) => {
    const index = users.value.findIndex(user => user.id === response.data.id);
    users.value[index] = response.data;
    $('#userFormModal').modal('hide');
  })
      .catch((error) => {
    console.log(error);
  })
      .finally(() => {
    form.value.resetForm();
  });
};

const handleSubmit = (values, actions) => {
  if (editing.value) {
    updateUser(values, actions);
  }  else {
    createUser(values, actions);
  }
};

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
    <div class="container-fluid">
      <button @click="addUser" type="button" class="mb-2 btn btn-primary">Add New User</button>
      <div class="card">
        <div class="card-body">
          <table class="table table-bordered">
            <thead>
              <tr>
                <th style="width: 10px;">#</th>
                <th>Name</th>
                <th>Email</th>
                <th>Registered</th>
                <th>Role</th>
                <th>Options</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(user,index) in users" :key="user.id">
                <td>{{ index + 1}}</td>
                <td>{{ user.name}}</td>
                <td>{{ user.email}}</td>
                <td>-</td>
                <td>-</td>
                <td>
                  <a href="#" @click.prevent="editUser(user)"><i class="fa fa-edit"></i></a>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal fade" id="userFormModal" data-backdrop="static" tabindex="-1" role="dialog"
       aria-labelledby="staticBackdropLabel" aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">
            <span v-if="editing">Edit User</span>
            <span v-else>Add New User</span>
          </h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <Form ref="form" @submit="handleSubmit" :validation-schema="editing ? editUserSchema : createUserSchema" v-slot="{ errors }" :initial-values="formValues">
          <div class="modal-body">
            <form autocomplete="off">
              <div class="form-group">
                <label for="name">Name</label>
                <Field name="name" type="text" class="form-control" :class="{ 'is-invalid':errors.name }" id="name"
                       aria-describedby="nameHelp" placeholder="Enter full name"/>
                <span class="invalid-feedback">{{ errors.name }}</span>
              </div>

              <div class="form-group">
                <label for="email">Email</label>
                <Field name="email" type="email" class="form-control" :class="{ 'is-invalid':errors.email }" id="email"
                       aria-describedby="nameHelp" placeholder="Enter full name"/>
                <span class="invalid-feedback">{{ errors.email }}</span>
              </div>
            </form>

            <div class="form-group">
              <label for="email">Password</label>
              <Field name="password" type="password" class="form-control" :class="{ 'is-invalid':errors.password }" id="password"
                     aria-describedby="nameHelp" placeholder="Enter password"/>
              <span class="invalid-feedback">{{ errors.password }}</span>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-primary">Save</button>
          </div>
        </Form>
      </div>
    </div>
  </div>

</template>

<script>
  export default {
  name: "ListUsers"
  }
</script>

<style scoped>

</style>