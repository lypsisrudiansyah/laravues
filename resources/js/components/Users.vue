<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users</h3>

                <div class="card-tools">
                  <button class="btn btn-success" @click="newModal">Add New <i class="fa fa-user-plus fa-fw"></i></button>
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
                      <th>Registered At</th>
                      <th>Modify</th>
                    </tr>
                  </thead>

                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at | myDate}}</td>
                      <td>
                        <a href="javascript:void(0)" @click="editModal(user)"><i class="fa fa-edit"></i></a>
                        |
                        <a href="javascript:void(0)" @click="deleteUser(user.id)"><i class="fa fa-trash red"></i></a>
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
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 v-show="editMode" class="modal-title" id="addNewLabel">Update User's Info</h5>
                <h5 v-show="!editMode" class="modal-title" id="addNewLabel">Add New</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editMode ? updateUser() : createUser()">
                <div class="modal-body">
                  <div class="form-group">
                    <input v-model="form.name" type="text" name="name" placeholder="Name"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                  </div>

                  <div class="form-group">
                    <input v-model="form.email" type="email" name="email" placeholder="Email Address"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                    <has-error :form="form" field="email"></has-error>
                  </div>

                  <div class="form-group">
                    <textarea v-model="form.bio" type="text" name="bio" placeholder="Short Bio for User(Optional)"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                    <has-error :form="form" field="bio"></has-error>
                  </div>

                  <div class="form-group">
                    <select v-model="form.type" type="text" name="type" placeholder="Type Address"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                      <option value="">Select User Role</option>
                      <option value="admin">Admin</option>
                      <option value="user">StandardUser</option>
                      <option value="author">Author</option>
                    </select>
                    <has-error :form="form" field="type"></has-error>
                  </div>

                  <div class="form-group">
                    <input v-model="form.password" type="password" name="password"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                    <has-error :form="form" field="password"></has-error>
                  </div>

                </div>
              
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                  <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                  <button v-show="!editMode" type="submit" class="btn btn-success">Create</button>
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
          /* Data Function Returning Object */
          return { 
            editMode: false,
            users: {},
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
          updateUser() {
            this.$Progress.start();
            this.form.put('/api/user/'+this.form.id)
            .then(() => {
                $('#addNew').modal('hide');
                Swal.fire(
                  'Updated',
                  'User\'s Info Successfully Updated.',
                  'success'
                )
                Fire.$emit('AfterCreated');
                this.$Progress.finish();
            })
            .catch(() => {
              // Fail
              this.$Progress.fail();
            })
            // console.log("Editing Data");
          },
          editModal(user) {
            this.editMode = true;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(user);
          },
          newModal() {
            this.editMode = false; 
            this.form.reset();
            $('#addNew').modal('show');
          },

          deleteUser(id) {
            Swal.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              icon: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
              if (result.value) {
                // Send Request to the server      
                this.form.delete('api/user/'+id).then(() => {
                  
                    Swal.fire(
                      'Deleted!',
                      'Your file has been deleted.',
                      'success'
                    )
                    Fire.$emit('AfterCreated');
                }).catch(() => {
                  Swal.fire("Failed!", "There Was Something Wrong", "warning");
                });
              }        
            })
          },
          
          loadUsers() {
            this.$Progress.start();
            axios.get('api/user').then(({data}) => ( this.users = data.data ));
            this.$Progress.finish();
          },
          
          createUser() { 
            this.$Progress.start();
            this.form.post('/api/user')
            .then(() => {
              /* this.form.post('/api/user')
                .then(({data}) => { console.log(data) }); */
              Fire.$emit('AfterCreated');
              $('#addNew').modal('hide');
              Toast.fire({
                icon: 'success',
                title: 'User Created Successfully'
              });
              this.$Progress.finish();
            })
            .catch(() => {
              this.$Progress.fail();
            });
          }
        },

        created() {
          this.loadUsers();
          Fire.$on('AfterCreated', () => {
            this.loadUsers();
          });
          // setInterval(() => this.loadUsers(), 3000);
        },

        mounted() {
            console.log('Component mounted.')
        }
    }
</script>
