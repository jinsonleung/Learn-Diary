Project：VUE3 todolist 笔记
Date from：Dec 2020
By：Jinson Liang

---

#### 1. 子组件使用父组件带参函数

##### 1.1 父组件（App.vue）代码

+ 注意：<Header :addTodo="addTodo" /> 父组件的属性绑定的值（如:addTodo要与子级props中的属性值一样

  > ```
  > <template>
  >   <div class="todo-container">
  >     <div class="todo-wrap">
  >       <Header :addTodo="addTodo" />
  >       <List :todos="todos" />
  >       <Footer />
  >     </div>
  >   </div>
  > </template>
  > 
  > <script lang="ts">
  > import { defineComponent, reactive, toRefs } from "vue";
  > import Footer from "./views/Footer.vue";
  > import Header from "./views/Header.vue";
  > import List from "./views/List.vue";
  > import { Todo } from "./types/todo";
  > 
  > export default defineComponent({
  >   name: "App",
  >   components: { Header, List, Footer },
  >   //1）数据使用数组来存储，每个数组是一个对象，由id、title、isCompleted组成
  >   //2）数据定义在父组件当中，子组件很方便即可访问到父组件的数据
  >   //3）使用接口来防止他人修改todos对象的属性类型
  >   setup() {
  >     const state = reactive<{ todos: Todo[] }>({
  >       todos: [
  >         { id: 1, title: "宝马", isCompleted: false },
  >         { id: 2, title: "奔驰", isCompleted: false },
  >         { id: 3, title: "本田", isCompleted: true },
  >         { id: 4, title: "奥迪", isCompleted: false },
  >         { id: 5, title: "中华", isCompleted: true },
  >         { id: 6, title: "比亚迪", isCompleted: true }
  >       ]
  >     });
  >     const addTodo = (todo: Todo) => {
  >       state.todos.unshift(todo);
  >     };
  >     return {
  >       ...toRefs(state),
  >       addTodo
  >     };
  >   }
  > });
  > </script>
  > ```

##### 1.2 子组件Header.vue代码

+ ==注意props参数的引用==

> ```
> <template>
>   <div class="todo-header">
>     <input
>       type="text"
>       placeholder="请输入任务并按回车确认"
>       @keyup.enter="add"
>       v-model="todoText"
>     />
>   </div>
> </template>
> 
> <script lang="ts">
> import { ref, defineComponent } from "vue";
> export default defineComponent({
>   name: "Header",
>   props: {
>     addTodo: {
>       type: Function,
>       required: true
>     }
>   },
>   setup(props) {
>     const todoText = ref("");
>     const add = () => {
>       const newTodoText = todoText.value;
>       if (!newTodoText.trim()) return;
>       const newTodo = {
>         id: Date.now(),
>         title: newTodoText.trim(),
>         isCompleted: false
>       };
>       props.addTodo(newTodo);
>       todoText.value = "";
>     };
>     return {
>       todoText,
>       add
>     };
>   }
> });
> </script>
> ```

#### 2. 使用父组件属性值更新子组件Checkbox值（这个问题报错）

##### 2.1 解决方法

> + 对于checkbox使用v-model方式绑定是否选中
> + 通过计算属性来设置todo元素的isCompleted状态

> <template>
>   <li
>     class="todo-item"
>     @mouseenter="switchBgColor(true)"
>     @mouseleave="switchBgColor(false)"
>     :style="{ color: newColor, background: newBgColor }">
>     <label>
>       <input type="checkbox" v-model="isCompleted" />
>       <span>{{ todo.title }}</span>
>     </label>
>     <button class="btn btn-danger" v-show="isShow" @click="del">删除</button>
>   </li>
> </template>
>
>
> <script lang="ts">
> import { ref, defineComponent, computed } from "vue";
> import { Todo } from "./types/todo";
> export default defineComponent({
>   name: "Item",
>   // props: ["todo","index"],
>   props: {
>     todo: {
>       type: Object as () => Todo,
>       required: true
>     },
>     index: {
>       type: Number,
>       required: true
>     },
>     delTodo: {
>       type: Function,
>       required: true
>     },
>     updateState: {
>       type: Function,
>       required: true
>     }
>   },
>   setup(props) {
>     const refData = ref(0);
>     const newColor = ref("black");
>     const newBgColor = ref("white");
>     const isShow = ref(false);
>     const switchBgColor = (flag: boolean) => {
>       if (flag) {
>         newColor.value = "white";
>         newBgColor.value = "#ff6633";
>         isShow.value = true;
>       } else {
>         newColor.value = "black";
>         newBgColor.value = "white";
>         isShow.value = false;
>       }
>     };
>     //删除数据
>     const del = () => {
>       console.log(props.index);
>       props.delTodo(props.index);
>     };
>     //选中/不选中计算属性
>     const isCompleted = computed({
>       get() {
>         return props.todo.isCompleted;
>       },
>       set(val: boolean) {
>         props.updateState(props.todo, val);
>       }
>     });
>
>     return {
>       refData,
>       switchBgColor,
>       newColor,
>       newBgColor,
>       isShow,
>       del,
>       isCompleted
>     }; 
>     }
>     });
>     </script>
> <style scoped>
> li {
>   list-style: none;
>   height: 36px;
>   line-height: 36px;
>   padding: 0 5px;
>   border-bottom: 1px solid #ddd;
> }
> li label {
>   float: left;
>   cursor: pointer;
> }
> li label li input {
>   vertical-align: middle;
>   margin-right: 6px;
>   position: relateive;
>   top: -1px;
> }
>
> li button {
>   float: right;
>   /* display: none; */
>   margin-top: 3px;
> }
>
> li:before {
>   content: initial;
> }
>
> li:last-child {
>   border-bottom: none;
> }
> </style>



##### 2.1 父组件（App.vue）代码如上

##### 2.2 第一层子组件（List.vue）代码

> ```
> <template>
>   <ul class="todo-main">
>     <Item v-for="todo in todos" :key="todo.id" :todo="todo" />
>   </ul>
> </template>
> 
> <script lang="ts">
> import { ref, defineComponent } from "vue";
> import Item from "./Item.vue";
> export default defineComponent({
>   name: "List",
>   components: { Item },
>   props: ["todos"],
>   setup() {
>     const refData = ref(0);
>     return {
>       refData
>     }
>   }
> });
> </script>
> ```

##### 2.3 第二层子组件（Item.vue）代码

>  ==**这里的v-model双向数据绑定报错，需要解决，<input type="checkbox" :value="todo.isCompleted" v-model="todo.isCompleted" />**==

> ```
> <template>
>   <li
>     class="todo-item"
>     @mouseenter="mouseHandler(true)"
>     @mouseleave="mouseHandler(false)"
>     :style="{color:ftColor, background: bgColor}"
>   >
>     <label>
>       <!-- <input type="checkbox" :value="todo.isCompleted" v-model="todo.isCompleted" /> 
>       这里的v-model动态双向数据报错
>       -->
>       <input type="checkbox" :value="todo.isCompleted" />
>       <span>{{ todo.title }}</span>
>     </label>
>     <button class="btn btn-danger" style="display: none">删除</button>
>   </li>
> </template>
> 
> <script lang="ts">
> import { ref, defineComponent } from "vue";
> import { Todo } from "../types/todo";
> 
> export default defineComponent({
>   name: "Item",
>   // props: ["todo"],
>   props: {
>     todo: Object as () => Todo
>   },
>   setup() {
>     const ftColor = ref("black");
>     const bgColor = ref("white");
>     const refData = ref(0);
>     const mouseHandler = (flag: boolean) => {
>       if (flag) {
>         ftColor.value = "white";
>         bgColor.value = "red";
>       } else {
>         ftColor.value = "black";
>         bgColor.value = "white";
>       }
>     };
>     return {
>       refData,
>       mouseHandler,
>       ftolor,
>       bgColor
>     };
>   }
> });
> </script>
> ```

#### 3. 源码

##### 3.1 App.vue

> <template>
>   <div class="todo-container">
>     <div class="todo-wrap">
>       <Header :addTodo="addTodo" />
>       <List :todos="todos" :delTodo="delTodo" :updateState="updateState" />
>       <Footer :todos="todos" :checkAll="checkAll" :clearAllCheck="clearAllCheck" />
>     </div>
>   </div>
> </template>
>
> <script lang="ts">
> import { ref, defineComponent, reactive, toRefs } from "vue";
> import Header from "./views/todolist/Header.vue";
> import List from "./views/todolist/List.vue";
> import Footer from "./views/todolist/Footer.vue";
> import { Todo } from "./views/todolist/types/todo";
>
> export default defineComponent({
>   name: "App",
>   components: { Header, List, Footer },
>   setup() {
>     const refData = ref(0);
>     //响应式对象数组
>     const state = reactive<{ todos: Todo[] }>({
>       todos: [
>         { id: 1, title: "奔驰", isCompleted: true },
>         { id: 2, title: "比亚迪", isCompleted: false },
>         { id: 3, title: "奔驰", isCompleted: false },
>         { id: 4, title: "奥迪", isCompleted: true },
>         { id: 5, title: "宝马", isCompleted: true },
>         { id: 6, title: "长安", isCompleted: false },
>         { id: 7, title: "福田", isCompleted: true },
>         { id: 8, title: "玛莎拉蒂", isCompleted: false },
>         { id: 9, title: "法拉利", isCompleted: true }
>       ]
>     });
>     //添加数据
>     const addTodo = (todo: Todo) => {
>       state.todos.unshift(todo);
>     };
>     //删除todo
>     const delTodo = (index: number) => {
>       state.todos.splice(index, 1);
>     };
>
>     //更新todo的isCompleted状态
>     const updateState = (todo: Todo, isCompleted: boolean) => {
>       todo.isCompleted = isCompleted;
>     };
>     //设置全选与全不选
>     const checkAll = (flag:boolean)=>{
>       state.todos.forEach(todo => {
>         todo.isCompleted = flag;
>       });
>     }
>     //删除被选中的todo
>     const clearAllCheck = ()=> {
>       state.todos = state.todos.filter((todo)=>(!todo.isCompleted));
>     };
>     return {
>       refData,
>       ...toRefs(state),
>       addTodo,
>       delTodo,
>       updateState,
>       checkAll,
>       clearAllCheck,
>     };
>   }
> });
> </script>
>
> <style scoped>
> .todo-container {
>   width: 600px;
>   margin: 0 auto;
> }
> .todo-container .todo-wrap {
>   padding: 10px;
>   border: 1px solid #ddd;
>   border-radius: 5px;
> }
> </style>

##### 3.2 Head.vue

> <template>
>   <div class="todo-header">
>     <input
>       type="text"
>       placeholder="请输入任务并按回车确认"
>       v-model="todoStr"
>       @keyup.enter="add"
>     />
>   </div>
> </template>
>
> <script lang="ts">
> import { ref, defineComponent } from "vue";
> import { Todo } from "./types/todo";
> export default defineComponent({
>   name: "Header",
>   props: {
>     addTodo: {
>       type: Function,
>       required: true
>     }
>   },
>   setup(props) {
>     const todoStr = ref("");
>     //涛加todo
>     const add = () => {
>       if (todoStr.value.trim() == "") return;
>       const text = todoStr.value;
>       const todo: Todo = {
>         id: Date.now(),
>         title: text,
>         isCompleted: false
>       };
>       props.addTodo(todo);
>       todoStr.value = "";
>     };
>     return {
>       todoStr,
>       add
>     };
>   }
> });
> </script>
>
> <style scoped>
> /* header */
> .todo-header input {
>   width: 560px;
>   height: 28px;
>   font-size: 14px;
>   border: 1px solid #ccc;
>   border-radius: 4px;
>   padding: 4px 7px;
> }
> .todo-header input:focus {
>   outline: none;
>   border-color: rgba(82, 168, 236, 0.8);
>   box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.75) 0 1px 2px
>     rgba(82, 168, 236, 0.6);
> }
> </style>

##### 3.3 List.vue

> <template>
>   <ul class="todo-main">
>     <Item
>       v-for="(todo, index) in todos"
>       :key="index"
>       :todo="todo"
>       :delTodo="delTodo"
>       :index="index"
>       :updateState="updateState"
>     />
>   </ul>
> </template>
>
> <script lang="ts">
> import { ref, defineComponent } from "vue";
> import Item from "../todolist/Item.vue";
> export default defineComponent({
>   name: "List",
>   components: { Item },
>   props: ["todos", "delTodo", "updateState"],
>   setup() {
>     const refData = ref(0);
>     return {
>       refData
>     };
>   }
> });
> </script>
>
> <style scoped>
> /* main */
> .todo-main {
>   margin-left: 0px;
>   border: 1px solid #ddd;
>   border-radius: 2px;
>   padding: 0px;
> }
> .todo-empty {
>   height: 40px;
>   line-height: 40px;
>   border: 1px solid #ddd;
>   border-radius: 2px;
>   padding-left: 5px;
>   margin-top: 10px;
> }
> </style>

##### 3.4 Item.vue

> <template>
>   <li
>     class="todo-item"
>     @mouseenter="switchBgColor(true)"
>     @mouseleave="switchBgColor(false)"
>     :style="{ color: newColor, background: newBgColor }"
>   >
>     <label>
>       <input type="checkbox" v-model="isCompleted" />
>       <span>{{ todo.title }}</span>
>     </label>
>     <button class="btn btn-danger" v-show="isShow" @click="del">删除</button>
>   </li>
> </template>
>
> <script lang="ts">
> import { ref, defineComponent, computed } from "vue";
> import { Todo } from "./types/todo";
> export default defineComponent({
>   name: "Item",
>   // props: ["todo","index"],
>   props: {
>     todo: {
>       type: Object as () => Todo,
>       required: true
>     },
>     index: {
>       type: Number,
>       required: true
>     },
>     delTodo: {
>       type: Function,
>       required: true
>     },
>     updateState: {
>       type: Function,
>       required: true
>     }
>   },
>   setup(props) {
>     const refData = ref(0);
>     const newColor = ref("black");
>     const newBgColor = ref("white");
>     const isShow = ref(false);
>     const switchBgColor = (flag: boolean) => {
>       if (flag) {
>         newColor.value = "white";
>         newBgColor.value = "#ff6633";
>         isShow.value = true;
>       } else {
>         newColor.value = "black";
>         newBgColor.value = "white";
>         isShow.value = false;
>       }
>     };
>     //删除数据
>     const del = () => {
>       console.log(props.index);
>       props.delTodo(props.index);
>     };
>     //选中/不选中计算属性
>     const isCompleted = computed({
>       get() {
>         return props.todo.isCompleted;
>       },
>       set(val: boolean) {
>         props.updateState(props.todo, val);
>       }
>     });
>
>     return {
>       refData,
>       switchBgColor,
>       newColor,
>       newBgColor,
>       isShow,
>       del,
>       isCompleted
>     };
>   }
> });
> </script>
>
> <style scoped>
> li {
>   list-style: none;
>   height: 36px;
>   line-height: 36px;
>   padding: 0 5px;
>   border-bottom: 1px solid #ddd;
> }
>
> li label {
>   float: left;
>   cursor: pointer;
> }
> li label li input {
>   vertical-align: middle;
>   margin-right: 6px;
>   position: relateive;
>   top: -1px;
> }
>
> li button {
>   float: right;
>   /* display: none; */
>   margin-top: 3px;
> }
>
> li:before {
>   content: initial;
> }
>
> li:last-child {
>   border-bottom: none;
> }
> </style>

##### 3.5 todo.ts

> //定义todo接口，目的是约束state数据类型
> export interface Todo {
>   id: number;
>   title: string;
>   isCompleted: boolean;
> }

##### 3.6 base.cs

> /* base */
> body {
>     background-color: #fff;
> }
>
> .btn {
>     display: inline-block;
>     padding: 4px 12px;
>     margin-bottom: 0;
>     line-height: 20px;
>     text-align: center;
>     vertical-align: middle;
>     cursor: pointer;
>     box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.2) 0 1px 2px rgba(0, 0, 0, 0.05);
>     border-radius: 4px;
> }
>
> .btn-danger {
>     color: #fff;
>     background-color: #da4f49;
>     border: 1px solid #bd362f;
> }
>
> .btn-danger:hover {
>     color: #fff;
>     background: #bd362f;
> }
>
> .btn:focus {
>     outline: none;
> }