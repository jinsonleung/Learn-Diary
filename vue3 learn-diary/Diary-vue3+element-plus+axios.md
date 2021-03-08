vue3+element-plus+axios

----

### 1. 获取组件对象

>         mounted(){
>             // 使用
>             const { proxy }:any = getCurrentInstance(); //获取上下文实例，ctx=vue2的this
>             proxy.axios.post('api/Login',{card:111}).then((e:any)=>{
>                 console.log(e)
>             })
>         },
> 