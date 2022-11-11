# Tips

1 所有 bug 第一时间检**查拼写错误**  
2 prop 数据可以加:形成动态绑定 v-bind，由父元素写在引用的子元素的前标签中，但是子元素只能读取数据，不能修改
3 通过 ref='name' 可以使 vue 像 css 中的 id 选择器一样标记一个组件，并通过 this.$ref.name.XXX 直接调用对应的组件实例
4

## 组件通信

1 prop 父传子 （传递为非函数变量）

        prop 数据可以加:形成动态绑定 v-bind，由父元素写在引用的子元素的前标签中，但是子元素只能读取数据，不能修改。（通常传递为非函数变量）

2 prop 子传父 （传递函数变量）

         由父组件在自组建的标签中声明数据名称，然后讲一个有参函数赋值给该数据，相当于通过prop 传递了一个函数给子组件。后由子组件对该函数进行调用，并将需要传递的数据data通过参数的方式传递给父组件。该函数应该对参数进行处理：如log显示，或赋值给父组件的某一个数据变量。

    父组件：

    <child :dataFromChild = "getData"></child>
        data(){
            return {
                dataFromChild:""
            }
        }
        methods:{
            getData(data){
                this.dataFromChild = data
            }
        }


    子组件：

    props:['dataFromChild']
    method:{
        sendData(){
            this.dataFromChild("**data**")
        }
    }
    **data** 为需要传递的数据

3 自定义事件 $on $emit 子传父  
( $emit 端发送数据， $on 端接收数据 )

在父组件中给子组件标签绑定 v-on：XXXEvent = “functionName”。 (@XXXEvent = “functionName)

在子组件中，通过 this.$emit('XXXEvent',data)传递数据。相当于同一个函数的执行和参数输入分别由两个组件控制，父组件负责执行，子组件负责传参数。

    父组件：
    <ChildrenCp v-on:childName="getName"><ChildrenCp>

     methods: {
        getName(data) {
            this.eName = data
        }
    }

    子组件：
    <button @click="sendName1">click event</button>
     methods: {
       sendName1() {
            this.$emit("childName", data)
        }
    }

3.1 通过 ref 改写 父组件部分
父组件：
<ChildrenCp ref="childCp"><ChildrenCp>

    mounted() {
        this.$ref.childCp.$on('childName',this.getName)

    }
     methods: {
        getName(data) {
            this.eName = data
        }
    }

    子组件：
    <button @click="sendName1">click event</button>
     methods: {
       sendName1() {
            this.$emit("childName", data)
        }
    }

3.2 通过事件总线 $bus  
 在 main.js 中 安装 $bus

    new Vue({
        el:'#app',
        render: h => h(App),
        beforeCreate() {
    	Vue.prototype.$bus = this //安装全局事件总线
    },
         })
    })

在同级兄弟组件中，需要**发送**数据的组件添加

    this.$bus.$emit('xxxEvent',data)

在同级兄弟组件中，需要**接收**数据的组件添加

    mounted() {
    		this.$bus.$on('xxxEvent',(data)=>{
    			console.log(data)
    		})
    	}

3.3 通过 $parent 或 $ root
与3.2基本相同，只是把$bus 换成了共有的祖先组件作为数据传递中间人  
传数据

    this.$parent.$emit('xxxEvent',data)

收数据

     mounted() {
    		this.$parent.$on('xxxEvent',(data)=>{
    			console.log(data))
    		})
    	}

4 provide 与 inject

在祖先组件定义 provide 属性，返回传递的值

    provide(){
    return {
        foo:'foo'
    }

}

在后代组件通过 inject 接收组件传递过来的值

    inject:['foo']

5 发布与订阅 （与 on emit 相似 ）

倒入 js 库 import pubsub from 'pubsub-js'

发送数据的组件

        // this.$bus.$emit('hello',this.name)
        pubsub.publish('hello',666)

接收数据的组件

    mounted() {
    		// console.log('School',this)
    		/* this.$bus.$on('hello',(data)=>{
    			console.log('我是School组件，收到了数据',data)
    		}) */
    		this.pubId = pubsub.subscribe('hello',(msgName,data)=>{
    			console.log(this)
    			// console.log('有人发布了hello消息，hello消息的回调执行了',msgName,data)
    		})
    	},
    	beforeDestroy() {
    		// this.$bus.$off('hello')
    		pubsub.unsubscribe(this.pubId) //取消订阅
    	},
    }

6 Vuex 万能

    通过引入Vuex并使用VueX插件.
    通常在src目录下创建一个store文件夹，在其index.js中引入插件并使用然后暴露store。

store/index.js （基础模版）

    import Vue from 'vue'
    //引入Vuex
    import Vuex from 'vuex'
    //应用Vuex插件
    Vue.use(Vuex)
    const actions = {

    }

    const mutations = {

    }
    const state = {
        sum:0 //当前的和
    }

    //创建并暴露store
    export default new Vuex.Store({
        actions,
        mutations,
        state,
    })

在 main.js 中，创建 Vue 根实例时将 store 加入配置项

    //引入store
    import store from './store'
    ····
        new Vue({
        el:'#app',
        render: h => h(App),
        store,          //添加store
        beforeCreate() {
            Vue.prototype.$bus = this
        }
    })

基本使用前提：

- mutations 中用来定义对数据的操作，只能为**同步**操作
- actions 中同样可以定义对数据的操作，可以有**异步**操作，但是是通过对 mutation 中的方法提交 commit 进行调用，间接实现对数据的操作
- state 中用于定义数据变量，不执行任何数据相关更新操作
- mutations 中的操作方法通过 this.$store.commit("xxx",date)对 xxx 方法进行调用
- actions 中的操作方法通过 this.$store.dispatch('xxx',data) 对 xxx 方法进行调用

使用实例：  
store/index.js

    const actions = {
        sayHi(context){
            setTimeout(() => {
                context.commit('sayHi')
            }, 5000);
        }
    }

    const mutations = {
            sayHi(state){
                console.log("hello"+state.myName);
            }
        }
    const state = {
            name:"lee"
        }

component.vue

    //调用 actions 中的 sayHi
    ...
    this.$store.dispatch("sayHi")

    // 调用 mutations 中的 sayHi
    ...
    this.$store.commit("sayHi")

### Tip

1 通过 引入 mapState 可以简化对 state 中数据的调用，在需要使用数据的组件 script 中加入以下：

    import {mapState,mapGetters} from 'vuex'
    ...
    computed: {
    ...mapState(['name'])
    }

此时引用 name 变量时，可以直接使用 name，而不是$store.state.myName

2 通过模块化和命名空间可以使分割 store，简化引用

## Vue-Router
