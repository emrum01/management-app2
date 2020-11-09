<template>
  <v-app>
  <div>
    <v-card-title>学習マネジメントアプリ</v-card-title>
    <p>ボタンを押すとモーダルウィンドウが開きます</p>
    <v-btn @click="openModal('category')">新しい問題を追加</v-btn>

    <!-- modal window群 -->
    <MyModal  v-show="modal.category" @close="closeModal('category')"> 
    
      <p>教科を入力してください</p>
      <div><input type="text" v-model="exercize.category" autofocus>
      </div>
    
        <button @click="closeModal('category');openModal('title')">送信</button>
    
    </MyModal>

    
    <MyModal @close="closeModal('title')" v-show="modal.title">
      
      <p>問題名を入力してください</p>
      <div><input type="text" v-model="exercize.title" autofocus>
      </div>
        <v-btn @click="closeModal('title');openModal('evaluation')">送信</v-btn>
      
    </MyModal>

    <MyModal @close="closeModal('evaluatoin')" v-show="modal.evaluation">
      
      <p>評価を入力してください</p>
      <div>
        <input type="radio" name="evaluate" value="得意" v-model="exercize.evaluation">できた！
        <input type="radio" name="evaluate" value="普通" v-model="exercize.evaluation">惜しかった！
        <input type="radio" name="evaluate" value="ニガテ" v-model="exercize.evaluation">不正解なり
      </div>
     
        <button @click="doAdd();closeModal('evaluation')">送信</button>
      
    </MyModal>

    <MyModal  v-show="modal.table" @close="closeModal('table')">
      
      <p>評価を入力してください</p>
      <div>
        <input type="radio" name="evaluate" value="得意" v-model="exercize.evaluation">できた！
        <input type="radio" name="evaluate" value="普通" v-model="exercize.evaluation">惜しかった！
        <input type="radio" name="evaluate" value="ニガテ" v-model="exercize.evaluation">不正解なり
      </div>
     
        <button @click="closeModal('table')"  >送信</button>
      
    </MyModal>
    <TodoTable 
      :todos="todos" 
      :headers="headers"
      @open-modal-from-table="saveIndex"
      @deleteInTable="doRemove"
    ></TodoTable>
    <GraphView :chart-data="datacollection"></GraphView>
    <!--
      @open-modal-from-table="openModal('table')" 
      @indexOfTodo="saveIndex" 
      @open-modal-from-table="openModal(modal.table)-->
  </div>
  </v-app>
</template>
<script src="https://cdn.jsdelivr.net/npm/axios@0.18.0/dist/axios.min.js"></script>
<script>
import MyModal from './components/MyModal.vue'
import TodoTable from './components/TodoTable.vue'
import GraphView from './components/BarChart.js'
import dayjs from 'dayjs';
import 'dayjs/locale/ja';
import dataOfStudent from './assets/data.json'

dayjs.locale('ja');
//import HelloWorld from './components/HelloWorld.vue'


export default {
  components: { MyModal,TodoTable,GraphView },
  //MyModal,TodoTable,
  data() {
    return {
      modal: {category:false,title:false,evaluation:false,table:false},
      message: '',
      exercize:{category:"",title:"",evaluation:"",date: undefined,howmany:0,score:0},
      //モック用のデータ
      todos:[],
      stuData: dataOfStudent,
      score: [],
      index: undefined,

      datacollection: null,

      headers: [
          { 
            text: '教科', 
            value: 'category' 
          },
          {
            text: '問題名',
            value: 'title',
          },
          { text: '状態', 
            value: 'evaluation' 
          },
          { text: '日付',
            value: 'date'
          },
          {
            text: 'やった回数',
            value: 'howmany'
          },
          {
            text:'done',
            value:'done'
          },
          {
            text:'削除',
            value:'delete',
            sortable:false
          }
        ]
    }
  },
  created(){
    this.todos = this.todos.concat(this.todos,this.stuData);
    this.saveTodo()
    for(let i=0; i<this.todos.length;i++){
      this.score.push(this.todos[i].score);
    }
  },
   mounted() {
    if (localStorage.getItem('todos')) {
      try {
        this.todos = JSON.parse(localStorage.getItem('todos'));
      } catch(e) {
        localStorage.removeItem('todos');
      }
    }
    confirm(this.score)
    this.fillData();
  },
  methods: {

    openModal(key) {
      this.modal[key]=true
    },
    closeModal(key) {
      if(key=='table'){
        this.todos[this.index].evaluation=this.exercize.evaluation
        this.todos[this.index].howmany += 1
        this.whenToDo()
      }
      this.modal[key] = false
      //this.exercize.evaluation="";
    },
    whenToDo(){
       //復習の回数によって次に学習する日付を決定
      switch(this.todos[this.index].howmany){
          case 2:
            this.todos[this.index].date=dayjs(this.todos[this.index].date).add(2, 'day').format('YYYY/MM/DD');
            break;
          case 3:
            this.todos[this.index].date=dayjs(this.todos[this.index].date).add(7, 'day').format('YYYY/MM/DD');
            break;
          case 4:
            this.todos[this.index].date=dayjs(this.todos[this.index].date).add(14, 'day').format('YYYY/MM/DD');
            break;
          case 5:
            this.todos[this.index].date=dayjs(this.todos[this.index].date).add(1, 'month').format('YYYY/MM/DD');
            break;
          case 6:
            this.todos[this.index].date=dayjs(this.todos[this.index].date).add(3, 'month').format('YYYY/MM/DD');
            break;
      }
    },
    doAdd(){
      this.exercize.date=dayjs().format("YYYY/MM/DD");
      this.exercize.howmany += 1;
      this.todos.push({...this.exercize});
      //initialize exercize
      this.exercize.category="";
      this.exercize.title="";
      this.exercize.evaluation="";
      this.exercize.howmany=0;
      //ローカルストレージに保存 
      this.saveTodo();
      return
    },
    doRemove: function(index) {
      this.todos.splice(index, 1);
      this.saveTodo();
    },
    saveTodo(){
      const parsed = JSON.stringify(this.todos);
      localStorage.setItem('todos', parsed);
    },
    changeEvaluationOfTodos(){
      this.todos[this.index].evaluation=this.exercize.evaluation;
    },
    saveIndex(number){
      this.index=number
      this.openModal('table')
    },
    //点数はイメージです
    fillData() {
      this.datacollection = {
        labels: ["A", "B", "C", "D"],
        datasets: [
          {
            label:"スコア",
            data: this.score
          }
        ]
      }
    }
  }
}
</script>