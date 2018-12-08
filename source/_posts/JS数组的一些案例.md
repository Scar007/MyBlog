---
title: JS数组中用的各种方法案例
subtitle: array
date: 2018-1-27
categories: JS
tags:
    - 数组
---
## 多维数组转一维数组
+ 方法一：使用数组concat方法，这个方法是属于取巧的一种
```javascript
var arr= [[0,0,1],[2,3,3],[4,4,5]];
var newArr = [];
for(var i=0;i<arr.length;i++){
     newArr=newArr.concat(arr[i])            
}
console.log(arr)  //[[0,0,1],[2,3,3],[4,4,5]];
console.log(newArr)  // [0, 0, 1, 2, 3, 3, 4, 4, 5]
```
+ 方法二：也是数组 join 方法，但是有一个缺点就是使数组每一项都变成了字符串
```javascript
var arr=[1,[2,[[3,4],5],6]];
function getArr(arr){ 
   return arr.join().split(","); 
} 
console.log(getArr(arr));
```
+ 方法三：递归
```javascript
var arr = [1,[2,[[3,4],5],6]];
var newArr = [];
    
function fun(arr){
        for(var i=0;i<arr.length;i++){
            if(Array.isArray(arr[i])){
                fun(arr[i]);
            }else{
                newArr.push(arr[i]);
            }
        }
    }
fun(arr);
console.log(newArr);//[1, 2, 3, 4, 5, 6]
```
+ 方法四：for in循环  递归
其实第四种跟第三种差不多，换用写法而已
```javascript
var arr =[1,[2,[[3,4],5],6]];
var newArr=[]; 
function getArr(arr) { 
    for(var k in arr) { 
       if( arr[k] instanceof Array) {
               getArr(arr[k]); 
     }else { newArr.push(arr[k]); 
   } 
  } 
  return newArr;
} 
console.log(getArr(arr));
```

## 一个数组中去除某一部分数组
```javascript
let arr = ['q', 'w', 'e', 'r', 't', 'y'];
let aaa = ['q', 'y'];
arr = arr.filter(item => !aaa.some(aaaItem => aaaItem === item));
console.log(arr)  //["w", "e", "r", "t"]
```

```javascript
let a = [{name:'gaojiarui',age:'18',id:'1231'},{name:'gjr',age:'16',id:'1324'},{namg:'gqjr',age:'17',id:'1234'}];
let b = [{name:'gjr',id:'1324'},{name:'gqjr',id:'1234'}]

a = a.filter(item => !b.some(aa => aa.id === item.id))
console.log(a);  //[{name: "gaojiarui", age: "18", id: "1231"}]
```

