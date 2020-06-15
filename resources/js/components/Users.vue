<template>
  <div class="container mt-5">
    <div class="row">
      <div class="col-md-12">
        <div class="card">
          <div class="card-header">
            <h3 class="card-title">Users</h3>

            <div class="card-tools">
              <button class="btn btn-success" @click="newModal">
                Add New
                <i class="fas fa-user-plus fa-fw"></i>
              </button>
            </div>
          </div>
          <!-- /.card-header -->
          <div class="card-body table-responsive p-0">
            <table class="table table-hover">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Name</th>
                  <th>E-mail</th>
                  <th>Type</th>
                  <th>Registered At</th>
                  <th>Modify</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="user in users" :key="user.id">
                  <td>{{ user.id }}</td>
                  <td>{{ user.name }}</td>
                  <td>{{ user.email }}</td>
                  <td>{{ user.type | upText }}</td>
                  <td>{{ user.created_at | myDate }}</td>
                  <td>
                    <span class="tag tag-success">Approved</span>
                  </td>
                  <td>
                    <button type="button" class="btn btn-primary" @click="editModal(user)">
                      <i class="fa fa-edit text-white"></i>
                    </button>
                    <button type="button" class="btn btn-danger" @click="deleteUser(user.id)">
                      <i class="fa fa-trash text-white"></i>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <!-- /.card-body -->
        </div>
        <!-- /.card -->
      </div>
    </div>
    <!-- Modal -->
    <div
      class="modal fade"
      id="addNew"
      tabindex="-1"
      role="dialog"
      aria-labelledby="addNewLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 v-show="!editmode" class="modal-title" id="addNewLabel">Add New User</h5>
            <h5 v-show="editmode" class="modal-title" id="addNewLabel">Update User Info</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <form @submit.prevent="editmode ? updateUser() : createUser()">
            <div class="modal-body">
              <div class="form-group">
                <input
                  v-model="form.name"
                  id="name"
                  type="text"
                  placeholder="Name"
                  name="name"
                  class="form-control"
                  :class="{
                                        'is-invalid': form.errors.has('name')
                                    }"
                />
                <has-error :form="form" field="name"></has-error>
              </div>
              <div class="form-group">
                <input
                  v-model="form.email"
                  id="email"
                  type="email"
                  placeholder="E-mail Address"
                  name="email"
                  class="form-control"
                  :class="{
                                        'is-invalid': form.errors.has('email')
                                    }"
                />
                <has-error :form="form" field="email"></has-error>
              </div>
              <div class="form-group">
                <textarea
                  v-model="form.bio"
                  type="text"
                  id="bio"
                  placeholder="Short Bio for user (optional)"
                  name="name"
                  class="form-control"
                  :class="{
                                        'is-invalid': form.errors.has('bio')
                                    }"
                />
                <has-error :form="form" field="bio"></has-error>
              </div>
              <div class="form-group">
                <select
                  v-model="form.type"
                  id="type"
                  name="type"
                  class="form-control"
                  :class="{
                                        'is-invalid': form.errors.has('type')
                                    }"
                >
                  <option value>Select user type</option>
                  <option value="admin">Admin</option>
                  <option value="user">Standard User</option>
                  <option value="author">Author</option>
                </select>
                <has-error :form="form" field="type"></has-error>
              </div>
              <div class="form-group">
                <input
                  v-model="form.password"
                  type="password"
                  placeholder="Password"
                  id="password"
                  name="password"
                  class="form-control"
                  :class="{
                                        'is-invalid': form.errors.has(
                                            'password'
                                        )
                                    }"
                />
                <has-error :form="form" field="password"></has-error>
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
              <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
              <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      editmode: true,
      users: {},
      form: new Form({
        id: "",
        name: "",
        email: "",
        password: "",
        type: "",
        bio: "",
        photo: ""
      })
    };
  },
  methods: {
    updateUser() {
      this.$Progress.start();
      this.form
        .put("api/user/" + this.form.id)
        .then(() => {
          Swal.fire("Updated!", "Current User data Updated", "success");
          this.$Progress.finish();
          $("#addNew").modal("hide");
          Fire.$emit("AfterCreate");
        })
        .catch(() => {
          this.$Progress.fail();
        });
    },
    editModal(user) {
      this.editmode = true;
      this.form.reset();
      $("#addNew").modal("show");
      this.form.fill(user);
    },
    newModal() {
      this.editmode = false;
      this.form.reset();
      $("#addNew").modal("show");
    },
    deleteUser(id) {
      Swal.fire({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!"
      }).then(result => {
        //send request to the server
        if (result.value) {
          this.form
            .delete("api/user/" + id)
            .then(() => {
              if (result.value) {
                Swal.fire("Deleted!", "User has been deleted.", "success");
              }
              Fire.$emit("AfterCreate");
            })
            .catch(() => {
              Swal("Failed!", "There was something wrong", "warning");
            });
        }
      });
    },
    loadUsers() {
      axios.get("api/user").then(({ data }) => (this.users = data.data));
    },
    createUser() {
      this.$Progress.start();
      this.form
        .post("api/user")
        .then(() => {
          Fire.$emit("AfterCreate");
          $("#addNew").modal("hide");
          Toast.fire({
            icon: "success",
            title: "User created successfully"
          });
          this.$Progress.finish();
        })
        .catch(() => {});
    }
  },
  created() {
    this.loadUsers();
    //setInterval(() => this.loadUsers(), 3000);
    Fire.$on("AfterCreate", () => {
      this.loadUsers();
    });
  }
};
</script>
