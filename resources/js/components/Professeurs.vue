<template>
    <div class="container">
       <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Liste des Professeurs </h3>

                <div class="card-tools">
                    <div class="input-group input-group-sm mx-md-n5" style="width: 250px;">
                        
                    <input type="text" name="table_search" class="form-control float-right" placeholder="Search">

                    <div class="input-group-append ">
                      <button type="submit" class="btn btn-default"><i class="fas fa-search"></i></button>
                    </div>
                    <div class="row m-2 "></div>
                                         <a class="btn btn-primary " @click="newModel()" href="#" role="button" data-toggle="modal" data-target="#addnew" >Add user <i class="fa fa-edit"/></a>

                  </div>


                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>name</th>
                      <th>Email</th>
                      <th>password</th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="prof in profs.data" :key="prof.id">
                      <td>{{prof.id}}</td>
                      <td>{{prof.name}} </td>
                      <td>{{prof.email}}</td>
                      <td>{{prof.password}}</td>

                      <td>
                          <center>
                          <a class="btn btn-success" href="#" role="button"  @click="editModel(prof)" >edit <i class="fa fa-edit"/></a>
                          <a class="btn btn-danger" href="#" @click="Deleteprof(prof.id)"  role="button">delete <i class="fa fa-edit"/></a>
                     </center>
                      </td>

                    </tr>

                  </tbody>
                </table>

              </div>
                <div class="card-footer">
                  <pagination :data="profs" @pagination-change-page="getResults"></pagination>
                </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>

                        <div class="modal fade" id="addnew" tabindex="-1" role="dialog" aria-labelledby="addnew" aria-hidden="true">
                <div class="modal-dialog  modal-dialog-centered" role="document">
                    <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="addnewTitle" v-show="!editmode" >add prof</h5>
                        <h5 class="modal-title" id="addnewTitle" v-show="editmode" >update prof</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                    </div>

                  <form @submit.prevent="editmode ? updateProf() : ajouteProf()">
                    <div class="modal-body">


                        <div class="form-group">
                        <input v-model="form.name" type="text" name="name"
                           placeholder="name" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                         </div>


                         <div class="form-group">
                        <input v-model="form.email" type="email" name="email"
                           placeholder="email" class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                         </div>

                          <div class="form-group">
                        <input v-model="form.password" type="password" name="password"
                           placeholder="password" class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                         </div>



                    </div>

                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" v-show="!editmode" class="btn btn-primary">create</button>
                        <button type="submit" v-show="editmode" class="btn btn-primary">updet</button>


                    </div>

                    </form>

                    </div>
                </div>
                </div>

    </div>
</template>

<script>
import Vue from 'vue'
import { Form, HasError, AlertError } from 'vform'
    export default {

     data() {
           return {
               editmode:false,
               profs:{},
      form: new Form({
        id:'',
        name: '',
        pernom: '',
        email: '',
        role:'1',
        password: '',

      })
    }
     },
     methods:{
          getResults(page = 1) {
                        axios.get('api/user?page=' + page)
                            .then(response => {
                                this.profs = response.data;
                            });
},
    loadProf(){
        axios.get("api/user").then((data)=>(this.profs = data))

    },

  ajouteProf(){
      this.$Progress.start();
      this.form.post("api/user")
    .then(() => {
        this.editmode=true;
   $('#addnew').modal('hide');
   Fire.$emit('AfterCreated');
   Toast.fire({
           icon: 'success',
           title: 'Signed in successfully'
           });
   this.$Progress.finish();})
   .catch(() => {})
    },



updateProf(){
    this.$Progress.start();
    this.form.put('api/user/'+this.form.id)
    .then((value) => {
       Fire.$emit('AfterCreated');
        $('#addnew').modal('hide');
          Swal.fire(
               'Updated!',
               'Your file has been updated.',
               'success'
                 )
   this.$Progress.finish();
    })
    .catch((err) => {
        this.$Progress.fail();
    })

},
  editModel(prof){
   this.editmode=true;
    this.form.reset();
   $('#addnew').modal('show');
   this.form.fill(prof);
},
  newModel(){
      this.editmode=false;
    this.form.reset();
   $('#addnew').modal('show');
},
  Deleteprof(id){
    Swal.fire({
       title: 'Are you sure?',
       text: "You won't be able to revert this!",
       icon: 'warning',
       showCancelButton: true,
       confirmButtonColor: '#3085d6',
       cancelButtonColor: '#d33',
       confirmButtonText: 'Yes, delete it!'
       }).then((result) => {
           if (result.value){
                    this.form.delete('api/user/'+id).then(()=>{
               Fire.$emit('AfterCreated');

               Swal.fire(
               'Deleted!',
               'Your file has been deleted.',
               'success'
                 )
              }
           ).catch((err) => {Swal.fire('failed !','Your file has been deleted.','error'
                 )})}

           }).catch((err) => {
               Swal.fire('failed !','Your file has been deleted.','success'
                 )
           })




},

     },
      created() {
    this.loadProf();
   Fire.$on('AfterCreated',()=>{
       this.loadProf();
    })
//  setInterval(()=>this.loadProf(),3000);
   },
     }

</script>
