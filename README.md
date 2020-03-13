# time-axis

> 一个时间轴选择插件

> [GitHub项目地址](https://github.com/HuXiaoTu/time-axis "GitHub")

> 效果演示: [https://github.com/HuXiaoTu/time-axis](https://github.com/HuXiaoTu/time-axis "GitHub")

> 欢迎留言沟通: 13676373110@163.com

## 使用

- ``` npm install time-axis ```  //安装包资源

### 全局注册方式
``` JavaScript
    //main.js中引入
    import TimeAxis from 'time-axis'
    Vue.use(time-axis)
```
``` JavaScript
    //*.vue中 直接使用
    <time-axis
        @getDateMessage="getDateMessage"
    >
    </time-axis>
    export default {
        data() {
            return {}
        },
        methods:{
            // 当选时间时
            getDateMessage(value){
                console.log(value)
            }
        }
    }
```

### 单独使用
``` JavaScript
    //*.vue中 直接使用
    <template>
        <time-axis
            @getDateMessage="getDateMessage"
        >
        </time-axis>
    </template>
    // 引入组件
    import TimeAxis from 'time-axis'
    export default {
        data() {
            return {}
        },
        methods:{
           // 当选时间时
            getDateMessage(value){
                console.log(value)
            }
        },
        // 注册组件
        components:{
            TimeAxis,
        }
    }
```

## Options && Methods

### Options

属性 | 说明 | 默认值 |
-|-|-
centerTime | 初始时间(时间轴最中间)(Date) | 当前时间 |
showTimeNumber | 总轴显示刻度个数(Number) | 24 |
dateType | 时间格式 | YYYY-MM-DD HH:mm |
regionalScope | 区域选中的范围(分钟)(Number) | 120 |
isShowTime | 拖动时是否实时展示选中时间(Boolean) | true |
isTimeBtn | 是否需要选择时间点按钮(Boolean) | true |
isTimeRegion | 是否需要选择时间区域(Boolean) | true |
TimeAxislength | 时间轴总时长(小时)(Number) | 24 |
leftRightLength | 左右切换时 切换的步长(小时)(Number) | 1 |
isAutoRefresh | 是否启动监听刷新(Boolean) | true |
autoRefreshMinute | 启动监听刷新间隔(分钟)(Number) | 1 |

### Methods

事件名称 | 说明 | 参数
-|-|-
getDateMessage | 当时间被选中是触发 | 当前选中的值(Object) |

## 二次开发

-  ``` npm install ```          //安装node运行环境

-  ``` npm run dev ```          //启动项目

-  ``` npm run build ```        //打包项目

## 仓库地址

GitHub项目地址[https://github.com/HuXiaoTu/time-axis](https://github.com/HuXiaoTu/time-axis "GitHub")


## 目录结构描述
```js
    ├─assets                    // 静态文件夹
    ├─css                       // 样式文件（字体图标）
    │  └─iconFont
    └─timeAxis                  // 组件开发区
```

