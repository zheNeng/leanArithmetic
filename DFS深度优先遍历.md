// 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移
// 动 一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4
// 矩阵中包含一条字符串“bcced”的路径，但是矩阵中不包含“abcb”路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该
// 格 子。
```js
class Path {
    constructor(coordinateRes, start, charsArr, contain) { //coordinateRes 已经走过的点,start 开始的点,charsArr 查找的字符串,contain 字符容器
        this.coordinateRes = coordinateRes || [] // 已经找过的
        this.coordinateRes.push(start) //将开始的点push到已经走过的点
        this.currentCoordinate = start // 当前的坐标
        this.contain = contain
        this.charsArr = charsArr // 
        this.currentChars = charsArr[coordinateRes.length] // 当前需要寻找的字符
        this.findStepLoop()
    }
    findStepLoop() {
        const loop = () => {
            const nextCoordinate = this.getNext(this.currentCoordinate, this.contain).filter(item => {
                const isRepeat = this.coordinateRes.some(isHas => this.U_eqArr(isHas, item)) // 坐标是否重复
                const isNextChars = this.currentChars == Path.findCharsFromContain(item, this.contain) // 该坐标是否是下一个字符的点
                return !isRepeat && isNextChars
            })
            const copy = { // 需要存下来，否则，下面forEach的时候，会改变 coordinateRes 的值
                coordinateRes: [...this.coordinateRes]
            }
            nextCoordinate.forEach((coordinate, index) => { // 隐藏bug，如果需要新建Path对象，需要放在push操作之前
                if (index >= 1) {
                    Path.callBackPush(new Path([...copy.coordinateRes], coordinate, this.charsArr, this.contain)) //创建一个新对象开启下一轮自举
                } else {
                    this.coordinateRes.push(coordinate) // 修改当前点的容器
                    this.currentCoordinate = coordinate // 修改当前点
                    this.currentChars = this.charsArr[this.coordinateRes.length] // 当前字符
                    if (this.currentChars) {
                        loop()
                    }
                }
            })
        }

        loop()
    }
    U_eqArr(a, b) {
        if (Object.prototype.toString.call(a) === '[object Array]' && Object.prototype.toString.call(b) === '[object Array]' && a.length === b.length) {
            for (let index in a) {
                if (a[index] !== b[index]) {
                    return false
                }
            }
            return true
        }
        return false
    }
    // 根据当前的坐标，查找下一个坐标
    getNext(coordinate, contain) {
        const length = coordinate.length
        const computedArr = ['add', 'subtract', 'same']
        const res = []
        function checkCoordinate(coordinate) { // 检查边界 和是否回退
            let chileContain = contain //存下引用
            for (let index in coordinate) {
                if (coordinate[index] < 0 || coordinate[index] > chileContain.length - 1) { //如果坐标小于0,或者大于最大坐标
                    return false
                }
                chileContain = chileContain[0]
            }
            return true
        }
        function checkRepeat(status, type) { //只允许上下左右运动，不允许运算重复
            return !status.some(item => {
                return item == type
            })
        }
        function loopGetNext(value, arr = [], status = []) {
            let currentLength = length - arr.length - 1

            for (let type of computedArr) {
                switch (type) {
                    case 'add':
                        const valueAdd = Number(value + 1)
                        if (currentLength > 0) {
                            loopGetNext(coordinate[arr.length + 1], [
                                ...arr,
                                valueAdd
                            ], [
                                    ...status,
                                    'add'
                                ])
                        } else {
                            const resArr = [
                                ...arr,
                                valueAdd
                            ]

                            if (checkCoordinate(resArr) && checkRepeat(status, 'add')) {
                                res.push(resArr)
                            }

                        }
                        break;
                    case 'subtract':
                        const valueSubtract = Number(value - 1)
                        if (currentLength > 0) {
                            loopGetNext(coordinate[arr.length + 1], [
                                ...arr,
                                valueSubtract
                            ], [
                                    ...status,
                                    'subtract'
                                ])
                        } else {
                            const resArr = [
                                ...arr,
                                valueSubtract
                            ]

                            if (checkCoordinate(resArr) && checkRepeat(status, 'subtract')) {
                                res.push(resArr)
                            }
                        }
                        break;
                    case 'same':
                        const valueSame = Number(value)
                        if (currentLength > 0) {
                            loopGetNext(coordinate[arr.length + 1], [
                                ...arr,
                                valueSame
                            ], [
                                    ...status,
                                    'same'
                                ])
                        } else {
                            const resArr = [
                                ...arr,
                                valueSame
                            ]

                            if (checkCoordinate(resArr) && checkRepeat(status, 'same')) {
                                res.push(resArr)
                            }
                        }
                        break;
                }
            }
        }
        loopGetNext(coordinate[0])
        //还需要滤过已经走过的点
        return res
    }
    // 根据字符找到坐标
    static findCoordinateFromContain(chars, contain = contain) {
        const res = []
        for (let i in contain) {
            for (let ii in contain[i]) {
                if (contain[i][ii] == chars) {
                    res.push([Number(i), Number(ii)])
                }
            }
        }
        return res
    }
    // 根据坐标找到字符
    static findCharsFromContain(coordinate, contain = contain) {
        try {
            return coordinate.reduce((a, b) => {
                return a[b]
            }, contain)
        } catch (err) {
            return null
        }
    }
}
function test(contain, str) {
    //1. 找到开始的点
    //2. 设置容器回调装 Path 对象
    const resPath = []
    const strArr = str.split('')
    const coordinateArr = Path.findCoordinateFromContain(strArr[0], contain)
    Path.callBackPush = function (item) {
        resPath.push(item)
    }
    for (let value of coordinateArr) {
        resPath.push(new Path([], value, strArr, contain))
    }
    const res = resPath.filter(item => {
        return item.coordinateRes && item.coordinateRes.length == strArr.length
    }).map(item => item.coordinateRes)
    console.log(JSON.stringify(res, null))
}
var contain = [
    [1, 2, 3],
    [6, 7, 8],
    [1, 2, 3],
    [6, 7, 8],
    [1, 2, 3],
    [6, 7, 8],
]
test(contain, '173')
```
