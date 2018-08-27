<template>
  <div class="calendar-box" id="app">
    <!--日历主体-->
    <div class="calendar-hd" :style="{background:'#'+hdBg}">
      <div class="calendar-date-box">
        <div class="calendar-date">
          <div class="calendar-date-month">{{selectMonth || currentMonth}}月</div>
          <div>
            <div class="calendar-day-year">{{selectYear || currentYear}}年</div>
            <div class="calendar-day-day">{{selectDay || currentDay}}日</div>
          </div>
        </div>
        <div class="calendar-today-button" @click="toToday">今</div>
      </div>
      <div class="calendar-week">
        <span>日</span><span>一</span><span>二</span><span>三</span><span>四</span><span>五</span><span>六</span>
      </div>
    </div>
    <div class="calendar-day-wrapper">
      <div ref="calendardaycontainer" class="calendar-day-container" v-vueswipe="swipeDo"
           style="left: -100vw;    transition:left 2s;">
        <div class="calendar-day-group" v-for="dateItem in dateArr">
                <span class="calendar-day-box otherMonth"
                      v-for="item in dateItem.total"
                      v-if="item <= dateItem.startWeek"
                      :class="{}"
                      @click="selectDate(1,dateItem.lastMonthDay-dateItem.startWeek+item)"><span>{{dateItem.lastMonthDay-dateItem.startWeek+item}}</span>
                </span>
          <span class="calendar-day-box"
                v-for="item in dateItem.total"
                v-if="item > dateItem.startWeek && item <= dateItem.startWeek+dateItem.fullDay"
                :class="{'today-flag':currentDay === item-dateItem.startWeek && currentMonth === selectMonth && currentYear === selectYear,'select-flag':selectDay === item-dateItem.startWeek}"
                @click="selectDate(2,item-dateItem.startWeek)"><span>{{item-dateItem.startWeek}}</span>
                </span>
          <span class="calendar-day-box otherMonth"
                v-for="item in dateItem.total"
                v-if="item > dateItem.startWeek+dateItem.fullDay"
                :class="{}"
                @click="selectDate(3,item-dateItem.startWeek-dateItem.fullDay)"><span>{{item-dateItem.startWeek-dateItem.fullDay}}</span>
                </span>
        </div>
      </div>
    </div>
    <!--日期选择主体-->
    <div class="mask">
      <div class="data-select-box">
        <div class="data-select-head">
          <div>取消</div>
          <div>确定</div>
        </div>
        <div class="data-select-content">
          <div class="data-select-value-box">
            <div class="data-select-group" v-vueswipe="selectSwipeDo">
              <span>2003 <span>年</span></span>
            </div>
            <div class="data-select-group">
              <span>03<span>月</span></span>
            </div>
            <div class="data-select-group">
              <span>03<span>日</span></span>
            </div>
          </div>
          <div class="content-group"><span v-for="item in 100">{{item+2000}}</span></div>
          <div class="content-group"><span v-for="item in 12">{{item < 10 ? '0'+item : item}}</span></div>
          <div class="content-group"><span v-for="item in 30">{{item < 10 ? '0'+item : item}}</span></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  /*
  * 触屏监听事件
  * */
  let vueTouch = function (el, binding, type) {//触屏函数
    let _this = this;
    this.obj = el;
    this.binding = binding;
    this.touchType = type;
    this.vueTouches = {x: 0, y: 0};//触屏坐标
    this.vueMoves = true;
    this.vueLeave = true;
    this.vueCallBack = function (e, disXY) {
      binding.value(el, e, disXY);
    };
    this.start = function (e) {//监听touchstart事件
      this.vueMoves = true;
      this.vueLeave = true;
      this.longTouch = true;
      this.vueTouches = {x: e.changedTouches[0].pageX, y: e.changedTouches[0].pageY};
      this.time = setTimeout(function () {
        if (this.vueLeave && this.vueMoves) {
          this.touchType === "vuelongtap" && this.vueCallBack(e);
          this.longTouch = false;
        }
      }.bind(this), 1000);
    };
    this.end = function (e) {//监听touchend事件
      let disX = e.changedTouches[0].pageX - this.vueTouches.x;//计算移动的位移差
      let disY = e.changedTouches[0].pageY - this.vueTouches.y;
      let disXY = {
        disX: disX,
        disY: disY
      };
      clearTimeout(this.time);
      if (Math.abs(disX) > 10 || Math.abs(disY) > 100) {//当横向位移大于10，纵向位移大于100，则判定为滑动事件
        this.touchType === "vueswipe" ? this.vueCallBack(e, disXY) : '';//若为滑动事件则返回
        if (Math.abs(disX) > Math.abs(disY)) {//判断是横向滑动还是纵向滑动
          if (disX > 10) {
            this.touchType === "vueswiperight" ? this.vueCallBack(e, disXY) : '';//右滑
          }
          if (disX < -10) {
            this.touchType === "vueswipeleft" ? this.vueCallBack(e, disXY) : '';//左滑
          }
        } else {
          if (disY > 10) {
            this.touchType === "vueswipedown" ? this.vueCallBack(e, disXY) : '';//下滑
          }
          if (disY < -10) {
            this.touchType === "vueswipeup" ? this.vueCallBack(e, disXY) : '';//上滑
          }
        }

      } else {
        if (this.longTouch && this.vueMoves) {
          this.touchType === "vuetap" ? this.vueCallBack(e) : '';
          this.vueLeave = false
        }
      }
    };
    this.move = function (e) {//监听touchmove事件
      this.vueMoves = false;
    };
    let EVENT_START, EVENT_MOVE, EVENT_END;
    if ('ontouchstart' in window) {
      EVENT_START = 'touchstart';
      EVENT_MOVE = 'touchmove';
      EVENT_END = 'touchend';
    } else {
      EVENT_START = 'mousedown';
      EVENT_MOVE = 'mousemove';
      EVENT_END = 'mouseup';
    }
    this.obj.addEventListener(EVENT_START, function (e) {
      _this.start(e);
    }, false);
    this.obj.addEventListener(EVENT_MOVE, function (e) {
      _this.end(e);
    }, false);
    this.obj.addEventListener(EVENT_END, function (e) {
      _this.move(e);
    }, false);
  };
  export default {
    name: "calendar",
    data() {
      return {
        windowWidth: '',
        hdBg: '04a3ee',
        curDate: '',
        year: '',
        month: '',
        day: '',
        dateArr: [],//日历数组
        currentYear: '',//当前时间
        currentMonth: '',
        currentDay: '',
        selectYear: '',//选择时间
        selectMonth: '',
        selectDay: '',
        trsXOld: '',//历史滚动幅度
        trsX: '',//日历滚动的幅度
        disX: '',//距离
        isSwipe: false,
      }
    },
    directives: {
      vuetap: {//点击事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vuetap");
        }
      },
      vueswipe: {//滑动事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswipe");
        }
      },
      vueswipeleft: {//左滑事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswipeleft");
        }
      },
      vueswiperight: {//右滑事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswiperight");
        }
      },
      vueswipedown: {//下滑事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswipedown");
        }
      },
      vueswipeup: {//上滑事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswipeup");
        }
      },
      longtap: {//长按事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "longtap");
        }
      }
    },
    mounted() {
      this.initData();
      this.swipeDone();
      this.windowWidth = document.body.clientWidth;
      this.trsXOld = this.windowWidth;
      this.trsX = this.windowWidth;
    },
    methods: {
      /*
      * 日历初始化
      * */
      initData(y, m, d) {
        if (!y) {
          let curDate = new Date();
          this.curDate = new Date();
          this.currentYear = curDate.getFullYear();
          this.currentMonth = curDate.getMonth() + 1;
          this.currentDay = curDate.getDate();
        } else {
          let fullDay = new Date(y, m + 1, 0).getDate();//当月总天数
          if (d > fullDay) {
            d = fullDay;
          }
          this.curDate = new Date(y, m, d);
        }
        this.initSelectDate(this.curDate);
        this.year = this.curDate.getFullYear();
        this.month = this.curDate.getMonth();
        this.dateArr = [];
        this.dateArr.push(this.mathDate(this.year, this.month - 1));
        this.dateArr.push(this.mathDate(this.year, this.month));
        this.dateArr.push(this.mathDate(this.year, this.month + 1));
        this.$refs.calendardaycontainer.style = '-100vw';
        this.trsXOld = this.windowWidth;
        this.trsX = this.windowWidth
      },
      mathDate(year, month) {
        let fullDay = new Date(year, month + 1, 0).getDate();//当月总天数
        let startWeek = new Date(year, month, 1).getDay();//当月第一天是周几
        let lastMonthDay = new Date(year, month, 0).getDate();//上月的总天数
        let total = (fullDay + startWeek) % 7 === 0 ? fullDay + startWeek : fullDay + startWeek + (7 - (fullDay + startWeek) % 7);//日历元素
        return {
          fullDay: fullDay,
          startWeek: startWeek,
          lastMonthDay: lastMonthDay,
          total: total,
        }
      },
      /*
      * 选中时间显示初始化
      * */
      initSelectDate(date) {
        this.selectYear = date.getFullYear();
        this.selectMonth = date.getMonth() + 1;
        this.selectDay = date.getDate();
      },
      /*
      *时间选择函数
      * @type {number} type 1:上月，2：本月，3：下月
      * @day {number} 选中的时间
      * */
      selectDate(type, day) {
        switch (type) {
          case 1:
            this.initData(this.year, this.month - 1, day);
            this.selectDay = day;
            this.mathToSixteen(2);
            break;
          case 2:
            this.selectDay = day;
            break;
          case 3:
            this.initData(this.year, this.month + 1, day);
            this.selectDay = day;
            this.mathToSixteen(1);
            break;
        }
      },
      /*
      * 根据日期修改颜色
      * */
      mathToSixteen(event) {
        let str = this.hdBg;
        let num = parseInt(str, 16);
        if (event === 1) {
          num = num - 64;
        } else {
          num = num + 64;
        }
        num = num.toString(16);
        if (num.length <= 5) {
          num = '0' + num;
        }
        num = num.slice(-6);
        this.hdBg = num;
      },
      /*
      * 跳转至今日事件
      * */
      toToday() {
        this.initData(this.currentYear, this.currentMonth - 1, this.currentDay);
        this.hdBg = '04a3ee';
      },
      /*
      * 滑动事件
      * */
      swipeDo(el, e, disXY) {
        let disX = disXY.disX;
        this.disX = disX;
        this.trsX = this.trsXOld;
        this.trsX = this.trsX - disX;
        if (this.trsX < 0) {
          this.trsX = 0;
        }
        if (this.trsX > 2 * this.windowWidth) {
          this.trsX = 2 * this.windowWidth;
        }
        el.style.left = '-' + this.trsX + 'px';
        this.isSwipe = true;
      },
      swipeDone() {
        let _this = this;
        //结束给其赋值
        let EVENT_END;
        if ('ontouchstart' in window) {
          EVENT_END = 'touchend';
        } else {
          EVENT_END = 'mouseup';
        }
        document.querySelector('.calendar-day-container').addEventListener(EVENT_END, function (e) {
          if (_this.isSwipe) {
            let disX = _this.disX;
            if (disX < 0) {
              if (Math.abs(disX) > _this.windowWidth / 3) {
                _this.trsX = 2 * _this.windowWidth;
                _this.trsXOld = _this.trsX;
                _this.$refs.calendardaycontainer.style.left = '-' + _this.trsX + 'px';
                _this.month = _this.month + 1;
                _this.initData(_this.year, _this.month, _this.selectDay);
                _this.mathToSixteen(1);
              } else {
                _this.trsX = _this.trsXOld;
                _this.$refs.calendardaycontainer.style.left = '-' + _this.trsX + 'px';
              }
            }
            if (disX > 0) {
              if (Math.abs(disX) > _this.windowWidth / 3) {
                _this.trsX = 0;
                _this.trsXOld = _this.trsX;
                _this.$refs.calendardaycontainer.style.left = '-' + _this.trsX + 'px';
                _this.month = _this.month - 1;
                _this.initData(_this.year, _this.month, _this.selectDay);
                _this.mathToSixteen(2);
              } else {
                _this.trsX = _this.trsXOld;
                _this.$refs.calendardaycontainer.style.left = '-' + _this.trsX + 'px';
              }
            }
            _this.isSwipe = false;
          }
        }, false);
      },
      selectSwipeDo(el,e,disXY){
        console.log(disXY);
      }
    },
  }
</script>

<style lang="stylus" scoped>
  @import "../assets/calendar.styl";
</style>
