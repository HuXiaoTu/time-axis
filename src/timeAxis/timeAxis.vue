<template>
    <div class="TimeAxisContent">
        <div id="TimeAxisContent"></div>
    </div>
</template>

<script>
import '../css/iconFont/iconfont.css';
import '../css/theme/index.css';
import dayjs from 'dayjs';
import { Notification } from 'element-ui';
export default {
    name: 'timeAxis',
    props: {
        // 初始时间
        centerTime: {
            type: Date,
            default: function () {
                return new Date();
            },
        },
        // 要展示几个时间
        showTimeNumber: {
            type: Number,
            default: 24
        },
        // 时间格式
        dateType: {
            type: String,
            default: 'HH:mm'
        },
        // 区域范围(分钟)
        regionalScope: {
            type: Number,
            default: 120
        },

        // 是否实时显示时间
        isShowTime: {
            type: Boolean,
            default: true
        },
        // 是否需要选择时间点按钮
        isTimeBtn: {
            type: Boolean,
            default: true,
        },
        // 是否需要选择时间区域
        isTimeRegion: {
            type: Boolean,
            default: true,
        },

        // 时间轴总时长(小时)
        TimeAxislength: {
            type: Number,
            default: 24,
        },
        // 左右切换时 切换的步长(小时)
        leftRightLength: {
            type: Number,
            default: 1,
        },
        // 是否启动监听刷新
        isAutoRefresh: {
            type: Boolean,
            default: true,
        },
        // 启动监听刷新间隔(分钟)
        autoRefreshMinute: {
            type: Number,
            default: 1,
        },
        // 是否显示刷新按钮
        isShowRefresh: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            // 开始时间
            getStartTime: null,
            // 结束时间
            getEndTime: null,

            // 递增递减时间
            leftRightTime: null,

            // 当前选中实况信息
            liveTIme: null,
            // 当前选中预报信息
            predictionTime: null,

            // 时间轴总时长(分钟)
            TimeAxislengthMinute: null,

            // 实况 实时显示盒子
            liveSpan: null,
            // 预报 实时显示左侧盒子
            predictionLeftSpan: null,
            // 预报 实时显示右侧盒子
            predictionRightSpan: null,

            // 区域范围(分钟)
            regionalScopeChange: null,
            // 当前时间(可变化)
            centerTimeChange: null,

            // 计算时间类型
            dateComputed: 'YYYY-MM-DD HH:mm',
            // 定时器
            autoRefreshHandel: null

        }
    },
    mounted() {
        // 只加载一次 初始数据
        this.initMountedData(this.centerTime);
        // 生成页面及事件
        this.init(this.centerTime);
        // 是否启动监听刷新
        if (this.isAutoRefresh) {
            this.openAutoRefresh(this.autoRefreshMinute);
        }
    },
    methods: {
        // 初始化
        init(initialTime) {
            // 容器盒子
            let box = document.getElementById('TimeAxisContent');
            // 数据初始化
            this.initData(initialTime);
            // 各个组件、按钮等初始化
            let { PositionBtn, positionRegion, positionLeftBtn, positionRightBtn, positionRegionBox, stretchBox, refreshBtn } = this.componentsInit(box);
            // 各类事件绑定
            this.methodsInit({ PositionBtn, positionRegion, positionLeftBtn, positionRightBtn, positionRegionBox, box, stretchBox, refreshBtn });

        },
        // 初始化数据
        initData(initialTime) {
            // 获取开始时间
            this.getStartTime = dayjs(initialTime).subtract(this.TimeAxislength / 2, 'hour').format(this.dateComputed);
            // 获取结束时间
            this.getEndTime = dayjs(initialTime).add(this.TimeAxislength / 2, 'hour').format(this.dateComputed);
        },
        // 各个小按钮组件等初始化
        componentsInit(box) {
            // 创建时间轴
            let TimeAxis = this.createTimeAxis(box);
            // 创建可移动按钮(实况)
            let PositionBtn = this.isTimeBtn ? this.createPositionBtn(box) : '';
            // 创建可移动区域(预报)
            let { positionRegion, positionRegionBox, stretchBox } = this.isTimeRegion ? this.createPositionRegion(box) : '';
            // 创建左右切换箭头
            let { positionLeftBtn, positionRightBtn } = this.createPositionLeftRight(box);
            // 创建刷新按钮
            let { refreshBtn } = this.isShowRefresh ? this.createRefreshBtn(box) : '';
            return { PositionBtn, positionRegion, positionLeftBtn, positionRightBtn, positionRegionBox, stretchBox, refreshBtn }
        },
        // 各类事件绑定
        methodsInit({ PositionBtn, positionRegion, positionLeftBtn, positionRightBtn, positionRegionBox, box, stretchBox, refreshBtn }) {
            let body = document.body;
            if (PositionBtn) {
                // 可移动按钮事件绑定
                this.PositionBtnMethods({ PositionBtn, body, box, positionRegionBox });
            }
            if (positionRegion) {
                // 可移动区域事件绑定
                this.PositionRegionMethods({ positionRegion, body, box, PositionBtn });
            }
            if (refreshBtn) {
                // 手动刷新按钮绑定事件
                this.manualRefresh(refreshBtn);
            }
            // 左右箭头按钮绑定事件
            this.leftRightMethods({ positionLeftBtn, positionRightBtn, box });
            // 区域拉伸事件绑定
            this.stretchBoxMethods({ stretchBox, positionRegionBox, body });
        },
        // 初始数据 只加载一次
        initMountedData(initialTime) {
            // 时间轴总时长(分钟)
            this.TimeAxislengthMinute = this.TimeAxislength * 60;
            // 区域范围(分钟)(可变化)
            this.regionalScopeChange = this.regionalScope;
            // 当前时间(可变化)
            this.centerTimeChange = { time: this.centerTime, left: this.TimeAxislength * 60 / 2 };
            // 设置当前选中实况信息
            this.liveTIme = {
                time: dayjs(initialTime).format(this.dateComputed),
                left: this.TimeAxislengthMinute / 2,
            };
            // 设置当前选中预报信息
            this.predictionTime = {
                startTime: dayjs(initialTime).format(this.dateComputed),
                endTime: dayjs(initialTime).add(this.regionalScopeChange, 'minute').format(this.dateComputed),
                left: this.TimeAxislengthMinute / 2
            };
            // 盒子的宽度
            let TimeAxisContentBox = document.querySelector('.TimeAxisContent');
            TimeAxisContentBox.style.width = this.TimeAxislengthMinute + 80 + 'px';
            let TimeAxisContent = document.getElementById('TimeAxisContent');
            TimeAxisContent.style.width = this.TimeAxislengthMinute + 'px';
        },
        /**
         * 绑定各类事件区域------------------------------------------------------↓-----------
         */
        // 可移动按钮事件绑定
        PositionBtnMethods({ PositionBtn, body, box, positionRegionBox }) {
            let that = this;
            // 启动移动监听
            PositionBtn.onmousedown = function (down) {
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);
                // 默认left值
                let defaultLeft = PositionBtn.style.left ? parseInt(PositionBtn.style.left) : 0;
                // 最后的left值
                let left = null;
                // 添加移动事件
                body.onmousemove = function (move) {
                    let X = move.clientX - down.clientX + defaultLeft;
                    // 确保在范围内进行拖动
                    // 移动处理函数
                    let moveHandel = function () {
                        PositionBtn.style.left = X + 'px';
                        // 实时展示当前时间
                        if (that.isShowTime) {
                            let time = dayjs(that.getStartTime).add(left, 'minute').format(that.dateType);
                            that.liveSpan.innerText = time;
                        }
                    }
                    if (positionRegionBox) {

                        if (X >= 0 && X <= that.centerTimeChange.left && X <= parseInt(positionRegionBox.style.left)) {
                            left = X;
                            moveHandel();

                        }
                    } else {
                        if (X >= 0 && X <= that.centerTimeChange.left && X <= parseInt(positionRegionBox.style.left)) {
                            left = X;
                            moveHandel();
                        }
                    }
                };;
                // 结束移动监听
                body.onmouseup = function () {
                    body.onmousemove = null;
                    body.onmouseup = null;
                    if (left != null) {
                        let time = dayjs(that.getStartTime).add(left, 'minute').format(that.dateComputed);
                        that.liveTIme = {
                            time: time,
                            left: left,
                        }
                        // 是否启动监听刷新
                        if (that.isAutoRefresh) {
                            that.openAutoRefresh(that.autoRefreshMinute);
                        }
                        // 抛出数据
                        that.$emit('getDateMessage', {
                            spotTime: that.liveTIme.time,
                            regionStartTime: that.predictionTime.startTime,
                            regionEndTime: that.predictionTime.endTime,
                            type: 'spot',
                        })
                    }
                }
            };

        },
        // 可移动区域事件绑定
        PositionRegionMethods({ positionRegion, body, box, PositionBtn }) {
            let that = this;
            let node = positionRegion.parentNode;
            // 启动移动监听
            node.onmousedown = function (down) {
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);
                // 默认left值
                let defaultLeft = node.style.left ? parseInt(node.style.left) : 0;
                // 最后的left值
                let left = null;
                // 添加移动事件
                body.onmousemove = function (move) {

                    let X = move.clientX - down.clientX + defaultLeft;
                    // 移动处理函数
                    let moveHandel = function () {
                        node.style.left = X + 'px';
                        // 实时展示当前时间
                        if (that.isShowTime) {
                            let timeStart = dayjs(that.getStartTime).add(left, 'minute').format(that.dateType);
                            let timeEnd = dayjs(that.getStartTime).add(left + that.regionalScopeChange, 'minute').format(that.dateType);

                            that.predictionLeftSpan.innerText = timeStart;
                            that.predictionRightSpan.innerText = timeEnd;
                        }
                    }
                    // 确保在范围内进行拖动
                    if (PositionBtn) {
                        if (X >= parseInt(PositionBtn.style.left) && X <= (box.offsetWidth - that.regionalScopeChange)) {
                            left = X;
                            moveHandel(left)
                        }
                    } else {
                        if (X >= 0 && X <= (box.offsetWidth - that.regionalScopeChange)) {
                            left = X;
                            moveHandel(left)
                        }
                    }
                };
                // 结束移动监听
                body.onmouseup = function () {
                    body.onmousemove = null;
                    body.onmouseup = null;
                    if (left != null) {
                        let timeStart = dayjs(that.getStartTime).add(left, 'minute').format(that.dateComputed);
                        let timeEnd = dayjs(that.getStartTime).add(left + that.regionalScopeChange, 'minute').format(that.dateComputed);
                        that.predictionTime = {
                            startTime: timeStart,
                            endTime: timeEnd,
                            left,
                        };
                        // 是否启动监听刷新
                        if (that.isAutoRefresh) {
                            that.openAutoRefresh(that.autoRefreshMinute);
                        }
                        // 抛出数据
                        that.$emit('getDateMessage', {
                            spotTime: that.liveTIme.time,
                            regionStartTime: that.predictionTime.startTime,
                            regionEndTime: that.predictionTime.endTime,
                            type: 'region',
                        })
                    }
                }
            };

        },
        // 左右箭头按钮绑定事件
        leftRightMethods({ positionLeftBtn, positionRightBtn, box }) {
            let that = this;
            // 左按钮点击事件
            positionLeftBtn.onclick = function () {
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);

                let liveTIme = JSON.parse(JSON.stringify(that.liveTIme));
                let predictionTime = JSON.parse(JSON.stringify(that.predictionTime));
                liveTIme.left = liveTIme.left + that.leftRightLength * 60;
                predictionTime.left = predictionTime.left + that.leftRightLength * 60;


                if (predictionTime.left + that.regionalScopeChange > box.offsetWidth) {
                    Notification({
                        title: "提示信息",
                        offset: 100,
                        type: "warning",
                        message: "超出范围",
                        duration: 500,
                        showClose: false
                    });
                    return
                };

                that.liveTIme = liveTIme;
                that.predictionTime = predictionTime;
                that.centerTimeChange.left += that.leftRightLength * 60;

                // 抛出数据
                that.$emit('getDateMessage', {
                    spotTime: that.liveTIme.time,
                    regionStartTime: that.predictionTime.startTime,
                    regionEndTime: that.predictionTime.endTime,
                    type: 'left',
                })

                box.innerHTML = '';
                let noTime = that.leftRightTime ? that.leftRightTime : that.centerTimeChange.time;
                let time = dayjs(noTime).subtract(that.leftRightLength, 'hour').format(that.dateComputed);
                that.leftRightTime = time;
                that.init(time);
            }
            // 右按钮点击事件
            positionRightBtn.onclick = function () {
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);

                let liveTIme = JSON.parse(JSON.stringify(that.liveTIme));
                let predictionTime = JSON.parse(JSON.stringify(that.predictionTime));
                liveTIme.left = liveTIme.left - that.leftRightLength * 60;
                predictionTime.left = predictionTime.left - that.leftRightLength * 60;

                if (liveTIme.left < 0) {
                    Notification({
                        title: "提示信息",
                        offset: 100,
                        type: "warning",
                        message: "超出范围",
                        duration: 500,
                        showClose: false
                    });
                    return
                };

                that.liveTIme = liveTIme;
                that.predictionTime = predictionTime;
                that.centerTimeChange.left -= that.leftRightLength * 60;

                // 抛出数据
                that.$emit('getDateMessage', {
                    spotTime: that.liveTIme.time,
                    regionStartTime: that.predictionTime.startTime,
                    regionEndTime: that.predictionTime.endTime,
                    type: 'right',
                })

                box.innerHTML = '';
                let noTime = that.leftRightTime ? that.leftRightTime : that.centerTimeChange.time;
                let time = dayjs(noTime).add(that.leftRightLength, 'hour').format(that.dateComputed);
                that.leftRightTime = time;
                that.init(time);
            }
        },
        // 区域拉伸事件绑定
        stretchBoxMethods({ stretchBox, positionRegionBox, body }) {
            let that = this;
            stretchBox.onmousedown = function (down) {
                down.stopPropagation();
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);
                let width = positionRegionBox.offsetWidth;
                let interval = null;
                body.onmousemove = function (move) {
                    interval = move.clientX - down.clientX;
                    let left = width + interval + that.predictionTime.left;
                    // 在时间轴的范围内
                    if (left <= that.TimeAxislengthMinute) {
                        // 实时改变宽度
                        positionRegionBox.style.width = width + interval + 'px';
                        // 重置默认值
                        that.predictionTime.endTime = dayjs(that.getStartTime).add(left, 'minute').format(that.dateComputed);
                        that.predictionRightSpan.innerText = dayjs(that.getStartTime).add(left, 'minute').format(that.dateType);
                    }
                }
                body.onmouseup = function (move) {
                    body.onmousemove = null;
                    body.onmouseup = null;
                    if (interval) {
                        that.regionalScopeChange = width + interval;
                        // 是否启动监听刷新
                        if (that.isAutoRefresh) {
                            that.openAutoRefresh(that.autoRefreshMinute);
                        }
                        // 抛出数据
                        that.$emit('getDateMessage', {
                            spotTime: that.liveTIme.time,
                            regionStartTime: that.predictionTime.startTime,
                            regionEndTime: that.predictionTime.endTime,
                            type: 'regionStretch',
                        })
                    }
                }

            }
        },
        // 手动刷新  监听事件
        manualRefresh(refreshBtn) {
            let box = document.getElementById('TimeAxisContent');
            let that = this;

            refreshBtn.onclick = function () {
                // 关闭刷新监听
                clearInterval(that.autoRefreshHandel);

                let newTime = new Date();
                // 当前时间(可变化)
                that.centerTimeChange = { time: newTime, left: that.TimeAxislength * 60 / 2 };
                // 设置当前选中实况信息
                that.liveTIme = {
                    time: dayjs(newTime).format(that.dateComputed),
                    left: that.TimeAxislengthMinute / 2,
                };
                // 设置当前选中预报信息
                that.predictionTime = {
                    startTime: dayjs(newTime).format(that.dateComputed),
                    endTime: dayjs(newTime).add(that.regionalScopeChange, 'minute').format(that.dateComputed),
                    left: that.TimeAxislengthMinute / 2
                };

                // 重启定时器
                that.openAutoRefresh(that.autoRefreshMinute);

                // 抛出数据
                that.$emit('getDateMessage', {
                    spotTime: that.liveTIme.time,
                    regionStartTime: that.predictionTime.startTime,
                    regionEndTime: that.predictionTime.endTime,
                    type: 'manuaRefresh',
                })

                box.innerHTML = '';
                that.init(newTime);
            }
        },
        // 自动刷新  监听事件
        openAutoRefresh(autoRefreshMinute) {
            let box = document.getElementById('TimeAxisContent');
            let that = this;

            this.autoRefreshHandel = setInterval(function () {
                let newTime = new Date();

                let liveTIme = JSON.parse(JSON.stringify(that.liveTIme));
                let predictionTime = JSON.parse(JSON.stringify(that.predictionTime));

                let getStartTime = dayjs(newTime).subtract(12, 'hour').format(that.dateComputed);

                liveTIme.time = dayjs(getStartTime).add(liveTIme.left, 'minute').format(that.dateComputed);
                predictionTime.startTime = dayjs(getStartTime).add(predictionTime.left, 'minute').format(that.dateComputed);
                predictionTime.endTime = dayjs(getStartTime).add(predictionTime.left + that.regionalScopeChange, 'minute').format(that.dateComputed);

                that.liveTIme = liveTIme;
                that.predictionTime = predictionTime;

                // 抛出数据
                that.$emit('getDateMessage', {
                    spotTime: liveTIme.time,
                    regionStartTime: predictionTime.startTime,
                    regionEndTime: predictionTime.endTime,
                    type: 'autoRefresh',
                })

                box.innerHTML = '';
                that.init(newTime);
            }, autoRefreshMinute * 60 * 1000);
        },
        /**
         * 生成各类组件、按钮区域-------------------------------------------------↓----------
         */
        // 创建时间轴
        createTimeAxis(box) {

            // 生成总轴
            let div = document.createElement('div');
            div.classList.add('timeAxis');
            box.appendChild(div);

            // 生成当前时间钮
            let nowTimeCircular = document.createElement('span');
            nowTimeCircular.classList.add('nowTimeCircular');
            nowTimeCircular.style.left = this.centerTimeChange.left + 'px';
            div.appendChild(nowTimeCircular);

            // 生成各个点的时间
            // 开始时间
            let getStartTime = this.getStartTime;
            // 结束时间
            let getEndTime = this.getEndTime;
            // 要显示几个
            let showTimeNumber = this.showTimeNumber;
            // 总长度
            let totalLength = this.TimeAxislengthMinute;
            // 根据总时长和当前时间和显示个数 计算出 各个点的时间
            let timeList = this.getTimeSlot({ getStartTime, totalLength, showTimeNumber, getEndTime });
            timeList.forEach((item) => {
                // 创建节点大盒子
                let nodeDiv = document.createElement('div');
                nodeDiv.classList.add('nodeDiv');
                nodeDiv.style.left = item.number + 'px';


                let nodeSpan = document.createElement('span');
                nodeSpan.innerText = item.time;

                nodeDiv.appendChild(nodeSpan);
                div.appendChild(nodeDiv);

                let nodeI = document.createElement('i');
                nodeI.style.left = nodeDiv.offsetWidth / 2 + 'px';
                nodeDiv.appendChild(nodeI);
            })
            return div;
        },
        // 创建可移动按钮
        createPositionBtn(box) {
            // 按钮
            let div = document.createElement('div');
            div.classList.add('iconfont', 'icon-position', 'positionBtn');
            div.style.left = this.liveTIme.left + 'px';

            // 是否实时显示当前的时间
            if (this.isShowTime) {
                let span = document.createElement('span');
                span.innerText = dayjs(this.liveTIme.time).format(this.dateType);
                div.classList.add('positionBtnTime');
                div.appendChild(span);
                this.liveSpan = span;
            }
            box.appendChild(div);

            let i = document.createElement('i');
            div.appendChild(i);
            return div;
        },
        // 创建可移动区域
        createPositionRegion(box) {
            // 区域大盒子
            let positionRegionBox = document.createElement('div');
            positionRegionBox.classList.add('positionRegionBox');
            positionRegionBox.style.left = this.predictionTime.left + 'px';
            positionRegionBox.style.width = this.regionalScopeChange + 'px';


            // 区域
            let positionRegion = document.createElement('div');
            positionRegion.classList.add('positionRegion');
            positionRegionBox.appendChild(positionRegion);

            // 拉伸边框
            let stretchBox = document.createElement('span');
            stretchBox.classList.add('stretchBox');
            positionRegion.appendChild(stretchBox);

            // 是否实时显示当前的时间
            if (this.isShowTime) {
                let leftSpan = document.createElement('span');
                leftSpan.innerText = dayjs(this.predictionTime.startTime).format(this.dateType);
                leftSpan.classList.add('positionRegionLeft');

                let rightSpan = document.createElement('span');
                rightSpan.innerText = dayjs(this.predictionTime.endTime).format(this.dateType);
                rightSpan.classList.add('positionRegionRight');

                positionRegionBox.appendChild(leftSpan);
                positionRegionBox.appendChild(rightSpan);
                this.predictionLeftSpan = leftSpan;
                this.predictionRightSpan = rightSpan;
            }

            box.appendChild(positionRegionBox);

            return { positionRegion, positionRegionBox, stretchBox };
        },
        // 创建左右切换箭头
        createPositionLeftRight(box) {
            let positionLeftBtn = document.createElement('div');
            positionLeftBtn.classList.add('iconfont', 'icon-left', 'positionLeftBtn');

            let positionRightBtn = document.createElement('div');
            positionRightBtn.classList.add('iconfont', 'icon-right', 'positionRightBtn');

            box.appendChild(positionLeftBtn);
            box.appendChild(positionRightBtn);

            return { positionLeftBtn, positionRightBtn }
        },
        // 创建刷新按钮
        createRefreshBtn(box) {
            let refreshBtn = document.createElement('span');
            refreshBtn.classList.add('refreshBtn', 'iconfont', 'icon-refresh');
            box.appendChild(refreshBtn);
            return { refreshBtn };
        },
        /**
        * 各类算法区域----------------------------------------------------------↓----------
        */
        // 根据总时长和当前时间和显示个数 计算出 各个点的时间
        getTimeSlot({ getStartTime, totalLength, showTimeNumber, getEndTime }) {
            // 每份的值(不取最后一份)
            let single = totalLength / (showTimeNumber);
            // 最终数据
            let arr = [];
            // 初始份数
            let flg = 0;

            for (let index = 1; index <= showTimeNumber - 1; index++) {
                let obj = {};
                flg += single;
                obj.number = flg;
                obj.time = dayjs(getStartTime).add(flg, 'minute').format(this.dateType);
                arr.push(obj);
            }
            arr.unshift({
                number: 0,
                time: getStartTime
            });
            arr.push({
                number: totalLength,
                time: getEndTime
            })
            return arr
        }
    },
}
</script>

<style lang="scss">
.TimeAxisContent {
    margin: 0 auto;
    position: relative;
    overflow: hidden;
    height: 85px;
    border-radius: 8px;
    background-color: rgb(255, 255, 255);
    border: 1px solid #ccc;
    #TimeAxisContent {
        position: absolute;
        bottom: 20px;
        left: 40px;
        // width: 1440px;
        height: 20px;
        z-index: 2;
        font-size: 0;
        // 可拖动按钮
        .positionBtn {
            position: absolute;
            left: 0;
            top: -35px;
            width: 1px;
            height: 35px;
            text-align: center;
            font-size: 30px;
            cursor: pointer;
            color: rgb(2, 58, 2);
            span {
                display: none;
                font-size: 12px;
                height: 20px;
                width: 120px;
                position: absolute;
                top: -12px;
                left: -60px;
                text-align: center;
            }
            i {
                display: inline-block;
                width: 1px;
                height: 10px;
                position: absolute;
                bottom: 0;
                left: 0;
                background-color: rgb(2, 58, 2);
                z-index: 3;
            }
        }
        .positionBtn::before {
            position: absolute;
            left: -14px;
            top: 0;
        }
        .positionBtn:hover {
            > span {
                display: inline-block;
            }
        }

        // 可拖动区域
        .positionRegionBox {
            height: 20px;
            position: absolute;
            left: 0;
            bottom: 10px;
            z-index: 3;
            font-size: 12px;
            color: rgb(2, 58, 2);
            .positionRegionLeft {
                display: none;
                position: absolute;
                width: 120px;
                height: 100%;
                left: -130px;
                top: -4px;
                text-align: right;
            }
            .positionRegionRight {
                position: absolute;
                display: none;
                width: 120px;
                height: 100%;
                right: -130px;
                top: -4px;
                text-align: left;
            }
            .positionRegion {
                width: 100%;
                height: 20px;
                position: absolute;
                left: 0;
                top: 0;
                background-color: rgba(45, 199, 84, 0.9);
                border-radius: 3px;
                cursor: pointer;
                .stretchBox {
                    display: none;
                    position: absolute;
                    right: -8px;
                    width: 8px;
                    border-left: 8px solid #ccc;
                    border-top: 10px solid transparent;
                    border-bottom: 10px solid transparent;
                    height: 20px;
                    cursor: e-resize;
                }
            }
        }
        .positionRegionBox:hover {
            .positionRegionLeft,
            .positionRegionRight {
                display: inline-block;
            }
            .positionRegion .stretchBox {
                display: inline-block;
            }
        }

        // 时间轴
        .timeAxis {
            position: absolute;
            width: 100%;
            height: 4px;
            background-color: rgb(1, 109, 1);
            .nowTimeCircular {
                width: 5px;
                height: 5px;
                position: absolute;
                transform-origin: center;
                transform: translateX(-2px);
                top: 0;
                background-color: red;
                border-radius: 50%;
            }
            .nodeDiv {
                position: absolute;
                top: 4px;
                transform: translate(-50%);
                text-align: center;
                width: 100px;
                i {
                    display: inline-block;
                    width: 1px;
                    height: 10px;
                    background-color: skyblue;
                    position: absolute;
                    top: -14px;
                }
                span {
                    display: inline-block;
                    padding-top: 5px;
                    font-size: 12px;
                }
            }
            :nth-child(2) {
                font-weight: 900;
            }
            :last-child {
                font-weight: 900;
            }
        }
        // 左右箭头按钮
        .positionLeftBtn,
        .positionRightBtn {
            width: 30px;
            height: 30px;
            position: absolute;
            top: -15px;
            color: rgb(1, 109, 1);
            font-size: 30px;
            cursor: pointer;
        }
        .positionLeftBtn {
            left: -30px;
        }
        .positionRightBtn {
            right: -30px;
        }
        // 刷新按钮
        .refreshBtn {
            position: absolute;
            left: -35px;
            top: -40px;
            display: inline-block;
            width: 20px;
            height: 20px;
            font-size: 18px;
            color: #023a02;
            cursor: pointer;
            font-weight: 800;
        }
    }
}
</style>
<style lang="scss">
.TimeAxisContent {
    // 禁止页面选中
    -moz-user-select: none; /*火狐*/
    -webkit-user-select: none; /*webkit浏览器*/
    -ms-user-select: none; /*IE10*/
    -khtml-user-select: none; /*早期浏览器*/
    user-select: none;
    div,
    p,
    span,
    ul,
    li {
        box-sizing: border-box;
    }
    ul,
    li {
        padding: 0;
        margin: 0;
        list-style: none;
    }
}
</style>