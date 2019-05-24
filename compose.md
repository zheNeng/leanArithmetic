# redux中间件
## 源码实现
```js
const fucArr = [
    function f1(next) {
        debugger
        return  function f11(action) {
            debugger
            setTimeout(() => {
                console.log(action++);
                next(action)
            }, 300)
        }
    },
    function f2(next) {
        debugger
        return  function f22(action) {
            debugger
            setTimeout(() => {
                console.log(action++);
                next(action)
            }, 200)
        }
    },
    function f3(next) {
        debugger
        return  function f33(action) {
            debugger
            setTimeout(() => {
                console.log(action++);
                next(action)
            }, 100)
        }
    },
]

var run = arr => {
    var reduceResult = arr.reduce((pre, next) => {
        debugger
        return (...arg) => pre(next(...arg))
    });
    debugger
    return reduceResult(function space(){});
}
// reduceResult 包装之后 执行顺序 f3(space)=>f2=>f1=>f11=>f22=>f33
run(fucArr)(1);
```