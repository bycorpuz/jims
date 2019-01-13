<template>
    <div class="container">
        
        
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()"> <!-- para dile ma view kung ang user is dile admin -->
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User's Information Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                    <download-excel
                        class   = "btn btn-success"
                        :data   = "json_data"
                        :fields = "json_fields"
                        name    = "users.xls">
                        Download <i class="fas fa-download fa-fw"></i>
                    </download-excel>
                </div>

    

              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover" id="userTable">
                  <tbody>
                    <tr>
                        <th>#</th>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Type</th>
                        <th>Registered at</th>
                        <th>Modify</th>
                    </tr>
                    <tr v-for="(user, index) in users.data" :key="user.id">
                        <td>{{index + 1}}</td>
                        <td>{{user.id}}</td>
                        <td>{{user.name}}</td>
                        <td>{{user.email}}</td>
                        <td>{{user.type | upText}}</td>
                        <td>{{user.created_at | createdDate}}</td>
                        <td>
                            <a href="#" @click="editModal(user)">
                                <i class="fa fa-edit text-blue"></i>
                            </a>
                            /
                            <a href="#" @click="deleteUser(user.id)">
                                <i class="fa fa-trash text-red"></i>
                            </a>
                        </td>
                    </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" v-show="!editmode" id="addNewLabel">Add New</h5>
                    <h5 class="modal-title" v-show="editmode" id="addNewLabel">Update User's Information</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editmode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name" id="name" placeholder="Name"                        
                                class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email" id="email" placeholder="Email Address"                        
                                class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                            <textarea v-model="form.bio" type="text" name="bio" id="bio" placeholder="Short bio for the user (Optional)"                        
                                class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                            </textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select v-model="form.type" name="type" id="type"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password" id="password" placeholder="Password"                        
                                class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="editmode" type="submit" class="btn btn-primary">Update</button>
                        <button v-show="!editmode" type="submit" class="btn btn-success">Create</button>
                    </div>
                </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        data(){
            return {
                editmode: false,
                users : {},
                form: new Form({
                    id : '',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : ''
                }),

                json_fields: {
                    'Complete name': 'name',
                    'Email': 'email',
                    'Register at': 'created_at',
                },
                json_data: [],
                json_meta: [
                    [
                        {
                            'key': 'charset',
                            'value': 'utf-8'
                        }
                    ]
                ],
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
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    swal(
                        'Updated!',
                        'Information has been updated.',
                        'success'
                    )
                    Fire.$emit('AfterUpdate');
                    $('#addNew').modal('hide');
                    this.$Progress.finish();
                })
                .catch(()=>{
                    this.$Progress.fail();
                })
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            deleteUser(id) {
                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {

                    //Send request to the Server
                    if (result.value) {
                        this.form.delete('api/user/'+id)
                        .then(()=>{
                            swal(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                            )
                            Fire.$emit('AfterDelete');
                        })
                        .catch(()=>{
                            swal("Failed", "There was something went wrong.","warning");
                        });
                    }
                })
            },
            loadUsers() {
                //para dile ma view kung ang user is dile admin
                if(this.$gate.isAdminOrAuthor()){
                    axios.get("api/user").then(({ data }) => (this.users = data) );
                    axios.get("api/usersAll").then(({ data }) => (this.json_data = data.data) );
                }
            },
            createUser() {
                this.$Progress.start();
                this.form.post('api/user')
                .then(()=>{
                    Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');
                    toast({
                        type: 'success',
                        title: 'User created successfully'
                    });
                    this.$Progress.finish();
                })
                .catch(()=>{

                })
            }
        },
        created() {
            this.loadUsers();
            Fire.$on(['AfterCreate','AfterDelete','AfterUpdate'], () => {
                this.loadUsers();
            });
            Fire.$on(['Searching'], () => {
                let query = this.$parent.search; //$parent is the app.js
                axios.get('api/findUser?q=' + query)
                .then((data)=>{
                    this.users = data.data;
                })
                .catch(()=>{

                })
            });
        }
    }
</script>