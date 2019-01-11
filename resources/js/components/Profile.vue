<style>
.widget-user-header{
    background-position: center center;
    background-size: cover;
}
</style>

<template>
    <div class="container">
        <div class="row">
            <div class="col-md-12 mt-3">
                <div class="card card-widget widget-user">
                    <!-- Add the bg color to the header using any of the bg-* classes -->
                    <div class="widget-user-header text-white" style="background-image:url('./img/user-cover.png');">
                        <h3 class="widget-user-username">{{this.form.name}}</h3>
                        <h5 class="widget-user-desc">{{this.form.bio}}</h5>
                    </div>
                    <div class="widget-user-image">
                        <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar">
                    </div>
                    <div class="card-footer">
                        <div class="row">
                        <div class="col-sm-4 border-right">
                            <div class="description-block">
                            <h5 class="description-header">3,200</h5>
                            <span class="description-text">SALES</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        <div class="col-sm-4 border-right">
                            <div class="description-block">
                            <h5 class="description-header">13,000</h5>
                            <span class="description-text">FOLLOWERS</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        <div class="col-sm-4">
                            <div class="description-block">
                            <h5 class="description-header">35</h5>
                            <span class="description-text">PRODUCTS</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        </div>
                        <!-- /.row -->
                    </div>
                </div>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header p-2">
                        <ul class="nav nav-pills">
                        <li class="nav-item"><a class="nav-link active" href="#settings" data-toggle="tab">Settings</a></li>
                        </ul>
                    </div><!-- /.card-header -->
                    <div class="card-body">
                        <div class="tab-content">
                            <div class="tab-pane active" id="settings">
                                <form class="form-horizontal">
                                <div class="form-group">
                                    <label for="inputName" class="col-sm-2 control-label">Name</label>

                                    <div class="col-sm-12">
                                        <input v-model="form.name" type="text" class="form-control" id="inputName" placeholder="Name"
                                        :class="{ 'is-invalid': form.errors.has('name') }">
                                        <has-error :form="form" field="name"></has-error>
                                    </div>
                                </div>
                                <div class="form-group">
                                    <label for="inputEmail" class="col-sm-2 control-label">Email</label>

                                    <div class="col-sm-12">
                                        <input v-model="form.email" type="text" class="form-control" id="inputEmail" placeholder="Email"
                                        :class="{ 'is-invalid': form.errors.has('email') }">
                                        <has-error :form="form" field="email"></has-error>
                                    </div>
                                </div>
                                <div class="form-group">
                                    <label for="inputBio" class="col-sm-2 control-label">Biography</label>

                                    <div class="col-sm-12">
                                        <textarea v-model="form.bio" class="form-control" id="inputBio" placeholder="Biography"
                                        :class="{ 'is-invalid': form.errors.has('bio') }">
                                        </textarea>
                                    </div>
                                    <has-error :form="form" field="bio"></has-error>
                                </div>
                                <div class="form-group">
                                    <label for="photo" class="col-sm-2 control-label">Profile Photo</label>

                                    <div class="col-sm-12">
                                    <input @change="updateProfile" type="file" class="form-input" name="photo">
                                    </div>
                                </div>
                                <div class="form-group">
                                    <label for="inputPassword" class="col-sm-4 control-label">Password (leave if not changing)</label>

                                    <div class="col-sm-12">
                                        <input v-model="form.password" type="text" class="form-control" id="inputPassword" placeholder="Password"
                                        :class="{ 'is-invalid': form.errors.has('password') }">
                                        <has-error :form="form" field="password"></has-error>
                                    </div>
                                </div>
                                <div class="form-group">
                                    <div class="col-sm-offset-2 col-sm-12">
                                    <button @click.prevent="updateInfo" type="submit" class="btn btn-primary">Update</button>
                                    </div>
                                </div>
                                </form>
                            </div>
                            <!-- /.tab-pane -->
                        </div>
                        <!-- /.tab-content -->
                    </div><!-- /.card-body -->
                    </div>
                </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                form: new Form({
                    id : '',
                    name : '',
                    email : '',
                    password : '',
                    //type : '',
                    bio : '',
                    photo : ''
                })
            }
        },
        methods: {
            getProfilePhoto() {
                let photo = (this.form.photo.length > 200) ? this.form.photo : 'img/profile/' + this.form.photo ;
                return photo;
            },
            updateInfo() {
                this.$Progress.start();
                if(this.form.password == ''){
                    this.form.password = undefined;
                }
                
                this.form.put('api/profile')
                .then(()=>{
                    Fire.$emit('AfterUpdate');
                    this.$Progress.finish();
                    swal(
                        'Updated!',
                        'Information has been updated.',
                        'success'
                    )
                })
                .catch(()=>{
                    this.$Progress.fail();
                });
            },
            updateProfile(e) {
                let file = e.target.files[0];
                let reader =  new FileReader();

                if (file['size'] < 2111775){ //check if file size is less than 2MB
                    
                    //changes to base 64
                    reader.onloadend = (file) => {
                        this.form.photo = reader.result;
                    }
                    reader.readAsDataURL(file);
                } else {
                    swal({
                        type: 'error',
                        title: 'Oops...',
                        text: 'Your are uploading a large file',
                    })
                }
            },
            getUserInfo() {
                return axios.get("api/profile").then(({ data }) => (this.form.fill(data)) );
            }
        },
        mounted() {
            console.log('Component mounted.')
        },
        created() {
            this.getUserInfo();
            Fire.$on(['AfterCreate','AfterDelete','AfterUpdate'], () => {
                this.getUserInfo();
            });
        }
    }
</script>
