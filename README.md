# time-shaft

> 一个时间轴选择插件

> [GitHub项目地址,'https://github.com/HuXiaoTu/time-axis'](https://github.com/HuXiaoTu/time-axis "GitHub")

> 欢迎留言沟通: 13676373110@163.com

## demo演示

![](https://github.com/HuXiaoTu/time-axis/blob/master/src/assets/time-axis.gif)

## 使用

- ``` npm i time-shaft ```  //安装包资源

### 全局注册方式
``` JavaScript
    //main.js中引入
    import TimeShaft from 'time-shaft'
    Vue.use(TimeShaft)
```
``` JavaScript
    //*.vue中 直接使用
    <time-shaft
        @getDateMessage="getDateMessage"
    >
    </time-shaft>
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
        <time-shaft
            @getDateMessage="getDateMessage"
        >
        </time-shaft>
    </template>
    // 引入组件
    import TimeShaft from 'time-shaft'
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
            TimeShaft,
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
isShowRefresh | 是否显示刷新按钮(Boolean) | true |

- isAutoRefresh 自动刷新详解 -- 自动刷新表示时间轴会随着设定的刷新间隔进行整个时间的更新，当切换总轴的左右按钮时将停止刷新
- isShowRefresh 刷新按钮详解 -- 当点击总轴 左右切换时间按钮时，自动更新时间将被关闭，点击刷新按钮，回到当前时间并开启自动刷新功能
### Methods

事件名称 | 说明 | 参数
-|-|-
getDateMessage | 当改变时间触发 | 当前时间(Object)


- getDateMessage参数详解 参数中含type字段 -- 可能出现的结果为
  spot(拖动点触发) region(拖动区域触发) regionStretch(拉伸区域触发) autoRefresh(定时刷新触发) left(时间轴左侧箭头触发 注:切换左右按钮时间不发
    生改变) right(时间轴右侧箭头触发 注:切换左右按钮时间不发生改变)

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

## V1.0.1 版本内容更新
1. 修复图标不显示问题 更新README
## V1.0.2 版本内容更新
1. 修复超出范围提示功能样式问题
## V1.0.4 版本内容更新
1. 增加刷新功能
2. 优化代码
## V1.0.5 版本内容更新
1. 修复图标不能正常显示问题
2. 优化插件代码，减少体积