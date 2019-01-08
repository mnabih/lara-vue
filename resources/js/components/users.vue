<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                  <button class="btn btn-success" @click="newModal">AddNew <i class="fa fa-user-plus  "></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users.data" :key='user.id'>
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | capitalize }}</td>
                    <td>{{user.created_at | formate }}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fa fa-trash red"></i>
                        </a>
                    </td>
                  </tr>
                 
                  </tbody>
                </table>
                
              </div>

                <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
                </div>
              
              <!-- /.card-body -->
            </div>
            <!-- /.card -->

          </div>
        </div>




        <!-- Modal -->
        <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
          <div class="modal-dialog" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 v-show="!editMode" class="modal-title" id="exampleModalLabel">New User</h5>
                <h5 v-show="editMode" class="modal-title" id="exampleModalLabel">Edit User</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>

              <form @submit.prevent="editMode?updateUser():addUser()">
                <div class="modal-body">
                    <!-- Bootstrap 4 -->
                    <div class="form-group">
                      <input v-model="form.name" type="text" name="name"
                        placeholder="Name" 
                        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                      <has-error :form="form" field="name"></has-error>
                    </div>

                    <div class="form-group">
                      <input v-model="form.email" type="email" name="email"
                        placeholder="Email"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                      <has-error :form="form" field="email"></has-error>
                    </div>
                    
                    <div class="form-group">
                      <textarea v-model="form.pio" type="text" name="pio"
                        placeholder="short pio for user(optional)"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('pio') }"></textarea>
                      <has-error :form="form" field="pio"></has-error>
                    </div>
                    
                    <div class="form-group">
                      <select v-model="form.type" type="text" name="type"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                      </select>
                      <has-error :form="form" field="type"></has-error>
                    </div>

                    <div class="form-group">
                      <input v-model="form.password" type="password" name="password"
                        placeholder="Password" 
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                      <has-error :form="form" field="password"></has-error>
                    </div>
                  
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                  <button v-show="!editMode" type="submit" class="btn btn-primary"> Create</button>
                  <button v-show="editMode" type="submit" class="btn btn-success"> Update</button>
                </div>
              </form>
            </div>
          </div>
        </div>
    </div>
</template>

<script>
    export default {
        data () {
          return {
            editMode:true,
            users:{},
            // Create a new form instance
            form: new Form({
              id:'',
              name: '',
              email: '',
              type: '',
              pio: '',
              photo: '',
              password: '',
              remember: false
            })
          }
        },
        methods:{
          getResults(page = 1) {
            axios.get('api/user?page=' + page)
              .then(response => {
                this.users = response.data;
              });
          },
          editModal(user){
            this.editMode = true

            this.form.reset()
            $('#exampleModal').modal('show')
            this.form.fill(user)

          },
          newModal(){
            this.editMode = false

            this.form.reset()
            $('#exampleModal').modal('show')

          },
          deleteUser(id){
            Swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                  }).then((result) => {


                    if (result.value) { // mean if press ok on alert

                      // send request to database
                      this.form.delete('api/user/' + id)

                      Swal(
                        'Deleted!',
                        'Your file has been deleted.',
                        'success'
                      )
                      // resend users request for update users after delete
                      Fire.$emit('afterCreate');

                    }
                  }).catch(()=>{
                      Swal(
                        'error!',
                        'Your file has not been deleted.',
                          'fail'
                      )
                  })
          },
          addUser(){
            // Submit the form via a POST request

                this.$Progress.start()
                this.form.post('api/user') // route name in api routes
                   
                   // if validation success
                  .then( () => { 
                      this.$Progress.finish()
                      toast({
                          type: 'success',
                          title: 'User Added successfully'
                        })

                      $('#exampleModal').modal('hide')
                      Fire.$emit('afterCreate');

                      // mine
                      //this.allUsers() // it works put Events are good for informing other components﻿

                     })

                      // if erorrs
                    .catch( () => { 

                    })

                
          },

          updateUser(){
            // Submit the form via a POST request

                this.$Progress.start()
                this.form.put('api/user/' + this.form.id) // route name in api routes
                                                          // need user id
                   
                   // if validation success
                  .then( () => { 
                      this.$Progress.finish()
                      toast({
                          type: 'success',
                          title: 'User Updated successfully'
                        })

                      $('#exampleModal').modal('hide')
                      Fire.$emit('afterCreate');

                      // mine
                      //this.allUsers() // it works put Events are good for informing other components﻿

                     })

                      // if erorrs
                    .catch( () => { 
                      this.$Progress.fail()
                    })

                
          },
          allUsers(){
            axios.get('api/user').then( ({data}) =>  (this.users = data) );
          }

        },        
        created(){
          this.allUsers();
          //setInterval(()=>this.allUsers(), 3000); //send request every 3 secounds

          Fire.$on('afterCreate',() => {this.allUsers()})
        },
        
    }
</script>
