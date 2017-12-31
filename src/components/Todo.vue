<template>
  <div>
    <nav class="navbar navbar-default">
      <div class="container-fluid">
        <a class="navbar-brand">
          Hai {{ profile.username }}, Welcome to Your To-Do List App <i class="glyphicon glyphicon-ok"></i>
        </a>
        <ul class="nav navbar-nav navbar-right">
          <router-link to="" class="navbar-brand">
            <span @click="doLogout">Log out</span>
          </router-link>
        </ul>
      </div>
    </nav>
    <div class="container">
      <div class="row">
        <div class="col-sm-6">
          <div class="panel panel-default">
            <div class="panel-heading">
              <h3>What Would You Do?</h3>
            </div>
            <div class="panel-body">
              <form v-on:submit.prevent="addTodo">
                <div role="form" style="display: block;">
                  <input class="form-control" placeholder="Title" v-model="Todo.todo">
                  <textarea class="form-control" placeholder="Description" v-model="Todo.description"></textarea>
                  <div class="form-group">
                    <input type="date" class="form-control" placeholder="Date" v-model="Todo.date">
                  </div>
                  <button class="btn btn-primary">Add Todo</button>
                </div>
              </form>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="panel panel-default">
            <div class="panel-heading">
              <h3>Your To Do List :  </h3>
            </div>
            <div class="panel-body" style="border-top: 1px solid #dddddd;" v-for="(Todo, index) in Todos">
              <h4 class="list-group-item-heading">
                Title : {{ Todo.todo }}
              </h4>
              <h4 class="list-group-item-text">Description : {{ Todo.description }}</h4>
              <h4><i class="glyphicon glyphicon-calendar"></i> {{ Todo.date }}</h4>
              <br>
              <div style="float:left;">
                <label >
                  <span class="btn btn-sm btn-primary">
                    <input type="checkbox" v-model="Todo.status" true-value="Completed"
                    false-value="Uncompleted" @click="editTodo(Todo._id)" class="custom-control-input"> {{ Todo.status }}
                  </span>
                </label>
              </div>
              <button style="float:right;" class="btn btn-md btn-danger" @click= "removeTodo(index, Todo._id)">Delete</button>
              <br><br>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data(){
      return {
        Todo:{ todo: '', description:'', date:'', status:'Uncompleted' },
        Todos:[],
        search:'',
        profile:[]
      }
    },

    created() {
      this.getProfile(),
      this.getTodo()
    },

    methods:{
      addTodo() {
        if (localStorage.getItem("token")) {
          this.$http.post('/post/todo', this.Todo)
          .then((response) => {
            this.Todos.push(this.Todo)
            this.Todo = {}
            this.getTodo()
          })
          .catch((err) => {
            console.log(err);
          })
        }else {
          swal({
            title: 'Ooops',
            text: `You don't Have Access! `,
            icon: 'error',
            button: 'OK'
          })
        }

      },

      getTodo() {
        this.$http.get('/todos')
        .then((response) => {
          this.Todos = response.data
        })
        .catch((err) => {
          console.log(err);
        })
      },

      removeTodo(index, _id){
        let self = this
        self.$swal({
          title: 'Are you sure?',
          text: 'You will not be able to recover this task!',
          icon: 'warning',
          buttons: true,
          buttons: ['No, keep it', 'Yes, delete it!']
        }).then((result) => {
          if (localStorage.getItem("token")) {
            if (result) {
              self.$http.delete('/delete/todo/'+_id)
              .then(() => {
                self.Todos.splice(index, 1)
              }).catch((err) => {
                console.log(err);
              })
            }
          }else {
            swal({
              title: 'Ooops',
              text: `You don't Have Access! `,
              icon: 'error',
              button: 'OK'
            })
          }
        }).catch((err) => {
          console.log(err);
        })
      },

      getProfile(){
        this.$http.get('/profile')
        .then((response) => {
          this.profile = response.data
        })
        .catch((err) => {
          console.log(err);
        })
      },

      editTodo(_id){
        this.$http.put('/edit/todo/'+_id)
        .then((response) => {
          console.log(response.data.updatedTodo.status);
        })
        .catch((err) => {
          console.log(err);
        })
      },

      doLogout(){
        let self = this
        FB.getLoginStatus(function(response) {
          if (response && response.status === 'connected') {
            FB.logout(function(response) {
              console.log('Succes Logout');
            });
          }
        })
        setTimeout(function(){
          self.$router.push('/')
        }, 500)

        localStorage.removeItem("token")

      }
    }
  }
</script>

<style scoped="" lang="css">
  .form-control { margin-bottom: 10px; }
  .navbar       { border-radius:0; }
</style>
