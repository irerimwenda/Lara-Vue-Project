<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users List</h3>

                <div class="card-tools">
                  <button class="btn btn-success btn-sm" @click="newModal">
                      <i class="fas fa-user-plus fa-fw"></i> Add New
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
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registered</th>
                      <th>Actions</th>
                    </tr>
                  </thead>

                  <tbody>
                    <tr v-for="user in users.data" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td><span class="tag tag-success">{{user.type | capitalize}}</span></td>
                      <td>{{user.created_at | cleanDate}}</td>
                      <td>
                          <a @click="editModal(user)">
                              <i class="fa fa-edit blue"></i>
                          </a>
                            
                          <a @click="deleteUser(user.id)" class="pl-3">
                              <i class="fa fa-trash red"></i>
                          </a>
                      </td>
                    </tr>
                  </tbody>

                </table>
              </div>
              <!-- /.card-body -->
              
              <div class="card-footer">
                  <pagination :data="users" 
                    @pagination-change-page="getResults">
                  </pagination>
              </div>

            </div>
            <!-- /.card -->
          </div>
        </div>

        <div class="row justify-content-center" v-else>
            <div class="col-md-10">
                <not-found-error></not-found-error>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewCenter" tabindex="-1" role="dialog" aria-labelledby="addNewCenterTitle" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 v-show="!editmode" class="modal-title" id="addNewCenterTitle">Add New</h5>
                <h5 v-show="editmode" class="modal-title" id="addNewCenterTitle">Update User Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>

            <form @submit.prevent="editmode ? updateUser() : createUser()">
                <div class="modal-body">

                    <div class="form-group">
                        <label>Name</label>
                        <input v-model="form.name" type="text" name="name" id="name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>

                    <div class="form-group">
                        <label>Email</label>
                        <input v-model="form.email" type="email" name="email" id="email"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>

                    <div class="form-group">
                        <label>Bio</label>
                        <textarea v-model="form.bio" name="bio" id="bio" placeholder="Short intro bio (Optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                        </textarea>
                        <has-error :form="form" field="bio"></has-error>
                    </div>

                    <div class="form-group">
                        <label>User Type</label>
                        <select name="type" v-model="form.type" id="type" class="form-control" :class="{
                            'is-invalid': form.errors.has('type') }">
                            <option value="">Select User Role</option>
                            <option value="admin">Admin</option>
                            <option value="user">User</option>
                            <option value="author">Author</option>
                        </select>
                        <has-error :form="form" field="type"></has-error>
                    </div>

                    <div class="form-group">
                        <label>Password</label>
                        <input v-model="form.password" type="password" name="password" id="password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                    </div>

                </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
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
                editmode: false,
                users : {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                })
            }
        },
        methods: {
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            updateUser() {
                this.$Progress.start();
                //console.log('Editing data');
                this.form.put('api/user/' + this.form.id)
                .then(() => {
                    //success
                    $('#addNewCenter').modal('hide');
                    Swal.fire(
                        'Updated!',
                        'Your file has been updated.',
                        'success'
                        )
                    Fire.$emit('afterCreated');
                    this.$Progress.finish();
                })
                .catch(() => {
                    //error
                    this.$Progress.fail();
                });
            },
            newModal() {
                this.editmode = false;
                this.form.reset();
                $('#addNewCenter').modal('show');
            },
            editModal(user) {
                this.editmode = true;
                this.form.reset();
                $('#addNewCenter').modal('show');
                this.form.fill(user);
            },
            deleteUser(id) {
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {

                        // Send request to the server
                        if (result.value) {
                        this.form.delete('api/user/' + id)
                        .then(() => {
                            Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                            )
                        Fire.$emit('afterCreated');
                        }).catch(() => {
                            Swal("Failed!", "There was something wrong.", "warning");
                        });
                        }
                        
                    })
            },
            loadUsers() {
                if(this.$gate.isAdminOrAuthor()) {
                    axios.get("api/user").then(({ data }) => (this.users = data));
                }
            },
            createUser() {
                this.$Progress.start();

                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('afterCreated');
                    $('#addNewCenter').modal('hide');

                    Toast.fire({
                        type: 'success',
                        title: 'User created successfully'
                        })
                    this.$Progress.finish();
                })
                .catch(() => {
                    
                });
                
            }
        },
        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                .then((data) => {
                    this.users = data.data
                })
                .catch(() => {

                });
            })
            this.loadUsers();
            Fire.$on('afterCreated', () => {
                this.loadUsers();
            });
            //setInterval(() => this.loadUsers(), 3000)
        }
    }
</script>
