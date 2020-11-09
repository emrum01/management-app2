<template>
    <v-container>
    <v-row>
      <v-col cols="10">
        <v-data-table v-show="todos"
          :headers="headers"
          :items="todos"
        >
        <template v-slot:item.done="{item}">
            <v-btn 
                small
                color="light-green"
                @click="openModal(item)"
                >
                done
            </v-btn>
        </template>
        <template v-slot:item.delete="{ item }">
          <v-btn
            small
            color="error"
            @click="deleteItem(item)"
          >
            delete
          </v-btn>
        </template>
        </v-data-table>
      </v-col>
      </v-row>
      {{test}}
    </v-container>
    
</template>

<script>
export default {
    name:"TodoTable",
    props:{
        todos: Array,
        headers: Array
    },
    data(){
        return {test: undefined}
    },
    methods:{
        openModal(item){
            let index = this.todos.indexOf(item)
            confirm(index)
            this.$emit("open-modal-from-table",index);
        },
        deleteItem(item){
            let index = this.todos.indexOf(item)
            confirm('ガチで削除しますか') 
            console.log(this.todos)
            this.$emit('deleteInTable',index)
        }
    }
}
</script>