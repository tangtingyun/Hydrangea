---
title: vue学习记录
date: 2019-11-09 22:28:16
tags: vue
---

- watch 
<!-- more -->
1. 组件的写法
```js
1.
new Vue({
    el: '#app',
    data:{
         value:'',
    },
    watch:{
        'value':{
            handler:'getData',
            immediate:true
        },
    },
    methods:{
         getData:function(){
             console.log('get data',this.value);
         }
    }
})
2.
new Vue({
    el: '#app',
    data:{
         obj:{
             name:''
         }
    },
    watch:{
         'obj':{
            handler:function(){},
            deep:true
        }
    },
    methods:{
         getData:function(){
             console.log('get data',this.value);
         }
    }
})
3.
 new Vue({
     el: '#app',
     data:{
         obj:{
             name:''
         }
     },
     watch:{
         'obj.name':function(value){
             console.log(value);
         }
     },
     methods:{
         getData:function(){
             console.log('get data',this.value);
         }
     }
})
```
2. [API的写法](https://cn.vuejs.org/v2/api/#vm-watch)
```js
vm.$watch( expOrFn, callback, [options] )
参数：
    {string | Function} expOrFn
    {Function | Object} callback
    {Object} [options]
        {boolean} deep
        {boolean} immediate
返回值：{Function} unwatch

vm.$watch('someObject'/'a.b.c',, callback, {
  deep: true,
  immediate: true
})
```