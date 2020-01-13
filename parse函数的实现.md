# parse函数的实现

完成如下的parse函数：
```js

function parse(object,parm){
    return parm.split(/\.|\[|\]/) // 用界定符分割字符串
        .filter(x=>x) // 过滤空元素(两个界定符在一起，分割会产生一个空位置)
        .reduce((a,b)=>a && a[b],object) // 数组聚合成一个值 reduce大展拳脚的场景
}


// 满足：
const object={
    b:{
        c:4
    },
    d:[{
        e:5
    },
    {
        e:6
    }]
}
parse(object,'b.c') // 4
parse(object,'d[0].e') // 5
parse(object,'d.1.e') // 6
parse(object,'a.b.c') // undefined

```


`