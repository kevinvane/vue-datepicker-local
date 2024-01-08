<template>
<div class="datepicker" :class="{'datepicker-range':range,'datepicker__clearable':clearable&&text&&!disabled}">
  <input readonly :value="text" :class="[show ? 'focus' : '', inputClass]" :disabled="disabled" :placeholder="placeholder" :name="name" v-if="type!=='inline'"/>
  <a class="datepicker-close" @click.stop="cls"></a>
  <transition name="datepicker-anim">
    <div class="datepicker-popup" :class="[popupClass,{'datepicker-inline':type==='inline'}]" tabindex="-1" v-if="show||type==='inline'">
      <template v-if="range">
        <vue-datepicker-local-calendar v-model="dates[0]" :left="true"></vue-datepicker-local-calendar>
        <vue-datepicker-local-calendar v-model="dates[1]" :right="true"></vue-datepicker-local-calendar>
      </template>
      <template v-else>
        <vue-datepicker-local-calendar v-model="dates[0]"></vue-datepicker-local-calendar>
      </template>
      <div v-if="showButtons" class="datepicker__buttons">
        <button @click.prevent.stop="cancel" class="datepicker__button-cancel">{{this.local.cancelTip}}</button>
        <button @click.prevent.stop="submit" class="datepicker__button-select">{{this.local.submitTip}}</button>
      </div>
    </div>
  </transition>
</div>
</template>

<script>
import VueDatepickerLocalCalendar from './VueDatepickerLocalCalendar.vue'
export default {
  name: 'VueDatepickerLocal',
  components: { VueDatepickerLocalCalendar },
  props: {
    name: [String],
    inputClass: [String],
    popupClass: [String],
    value: [Date, Array, String],
    disabled: [Boolean],
    type: {
      type: String,
      default: 'normal'
    },
    rangeSeparator: {
      type: String,
      default: '~'
    },
    clearable: {
      type: Boolean,
      default: false
    },
    placeholder: [String],
    disabledDate: {
      type: Function,
      default: () => {
        return false
      }
    },
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    local: {
      type: Object,
      default () {
        return {
          dow: 1, // Monday is the first day of the week
          hourTip: '选择小时', // tip of select hour
          minuteTip: '选择分钟', // tip of select minute
          secondTip: '选择秒数', // tip of select second
          yearSuffix: '年', // format of head
          monthsHead: '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_'), // months of head
          months: '一月_二月_三月_四月_五月_六月_七月_八月_九月_十月_十一月_十二月'.split('_'), // months of panel
          weeks: '一_二_三_四_五_六_日'.split('_'), // weeks
          cancelTip: '取消', // default text for cancel button
          submitTip: '确定' // default text for submit button
        }
      }
    },
    showButtons: {
      type: Boolean,
      default: false
    },
    dateRangeSelect: [Function]
  },
  data () {
    return {
      show: false,
      dates: this.vi(this.value)
    }
  },
  computed: {
    range () {
      return this.dates.length === 2
    },
    text () {
      const val = this.value
      const txt = this.dates.map(date => this.tf(date)).join(` ${this.rangeSeparator} `)
      if (Array.isArray(val)) {
        return val.length > 1 ? txt : ''
      } else {
        return val ? txt : ''
      }
    }
  },
  watch: {
    value (val) {
      this.dates = this.vi(this.value)
    }
  },
  methods: {
    get () {
      return Array.isArray(this.value) ? this.dates : this.dates[0]
    },
    cls () {
      this.$emit('clear')
      this.$emit('input', this.range ? [] : '')
    },
    vi (val) {
      if (Array.isArray(val)) {
        return val.length > 1 ? val.map(item => new Date(item)) : [new Date(), new Date()]
      } else {
        return val ? new Array(new Date(val)) : [new Date()]
      }
    },
    ok (leaveOpened) {
      const $this = this
      $this.$emit('input', $this.get())
      !leaveOpened && !$this.showButtons && setTimeout(() => {
        $this.show = $this.range
      })
    },
    tf (time, format) {
      const year = time.getFullYear()
      const month = time.getMonth()
      const day = time.getDate()
      const hours24 = time.getHours()
      const hours = hours24 % 12 === 0 ? 12 : hours24 % 12
      const minutes = time.getMinutes()
      const seconds = time.getSeconds()
      const milliseconds = time.getMilliseconds()
      const dd = t => ('0' + t).slice(-2)
      const map = {
        YYYY: year,
        MM: dd(month + 1),
        MMM: this.local.months[month],
        MMMM: this.local.monthsHead[month],
        M: month + 1,
        DD: dd(day),
        D: day,
        HH: dd(hours24),
        H: hours24,
        hh: dd(hours),
        h: hours,
        mm: dd(minutes),
        m: minutes,
        ss: dd(seconds),
        s: seconds,
        S: milliseconds
      }
      return (format || this.format).replace(/Y+|M+|D+|H+|h+|m+|s+|S+/g, str => map[str])
    },
    dc (e) {
      this.show = this.$el.contains(e.target) && !this.disabled
    },
    submit () {
      this.$emit('confirm', this.get())
      this.show = false
    },
    cancel () {
      this.$emit('cancel')
      this.show = false
    }
  },
  mounted () {
    document.addEventListener('click', this.dc, true)
  },
  beforeDestroy () {
    document.removeEventListener('click', this.dc, true)
  }
}
</script>

<style>
.datepicker {
  display: inline-block;
  position: relative;
  width: 208px;
}

/* 日历图标 */
.datepicker:before {
  content: '';
  display: block;
  position: absolute;
  width: 34px;
  height: 34px;
  /* height: 100%; */
  top: 4px;
  left: 12%;
  /* right: 0; */
  /* background: url('./select_date_30.png')no-repeat 50% 50%; */
  /* 30*30 */
  background: url('data:image/png;base64,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')no-repeat 50% 50%;
  /* 20*20 */
  /* background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAVCAYAAABG1c6oAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAEgUlEQVQ4jW2UzWuc1xXGf+fe92O+NR7JHilyVVv5cCI14NiGNKUpKYVCIItAN6WBZNdN6aKQXRYJXZR2GyhkkSzyH3TZQkJpSSFtRVLhsVWKK9uJJTmS51PSzLx633tOFrIcUXpXBw7Pj3sennOE//PG4/HS7Xv3fjkZjcYVeG/l2rWdU73F7e7e27tbO4+VRN668vzz66e17n9hZlYaT7M3B73u6+WNjZ+H6zd+tba2Fp/0j0J4pdfr/Sy9ufEDXfvsF51OJzmtl5Ni73cf1JN4+IwsnF8cfv+F395d+2d3cb2T9ceTi3Vnvz5XKk0TiXz22k/fWN/e+m77j3/6ojcYlKvOvV8timHvzNnrZ1584dMIwN59Nw2N6DeTzeHrEoLX/e+Ubef+8oP/3rIkirwvl3+fmyE+Et17EIWj3G33+pcq+/ui3r0zHI6sPxodZKl/53jkajUJw9GFkGUNS9NqXKu7bGkp2ltdiY+GQzcZDkvT/qCkGlJmWz5tNGTn2lV3WBSSHxxG08k41uyoWUwmF4+Bo5FHLAZDSyV8mnBmYYHy3BwmYEAwRZ0jeE+z0WB2fh6JI8QJagYioEgEQBx7KZdLFkWwtYX//HPqIWCdDvVGgzRJSeIIm06RTz6hMj/PmbU1Kt6T1mpwOOYAQJUI4HB1NU5LpYrMzKAh4PoD6vkR9StXiJIYxCEiIIIAlX6fb126hF9dRUPgYHsb7fXQcvkYWH3ppaQoihorK0gIYEZqitlJFuQ4D2ZgRmTgRVARCIH8/g7ZV7tImh4DgdREaiKC5Ef4e7exnbsUPiZffgY/e44omyI3P8OKHAyGpTrx08+Cc6gaBphzj4AVVBsC+Fsd8r99jPmIKE0YbNwgfvkntJwxXf87+4dTXFriS0lpzLZpnWsTVDHsGw+BWYMqgC1eZHxhlXjuLMn929C5Ttbtgi/o7vboXnqOheeucr5SJWo0ECBo4MSdk9VbUDVvZmhzlvqPXqbWbBD2R4zNk9RqMOxTazVpp0K5UqY2d5Y0SUGgeOj7I2Ch+lTQ4ngNTXFbm+T/+As7m3ex7/2QartNXm1S/vYTtDVDP/oDe3c2UTPUjBBO/dDMnJg9qUEhBPwXtwif/pkvN/7DdPUKZy9fxTmHPf40/sUf485fwGdTdv99g6LIMVVCCMfpdxAB5aB6wVTx23dg7a/k3QdUWi1qKejNf+G8YYM+1BoU23foPegiT1wGEYIqRdBHDkZZlj1mxjk1w03GHEkEc20aZmhvj8HOLtHjyyTjA/L79xiMDhgsr7D47GUi5xkXU4KGR3cr8mm6NDk8bJoZxdKTaKuN6cMRzGhGEVSqhOmYsL9PIsL55ixJpYyqkYcCMxCOtyhyqk8BDUTQJEFas8iJw6diENISbqZF6eQQA0bBUZ4jIjjvcCIWqeqMOGfivAqGqQnuoeLkBBuYmXyDAjHDDPIQrFGrTcvlyghhUyZmy5YVryo6b0gSQkhBExEXm1qCkGCWArEhqRMSxMeCxUUoov2Dg/E4yz+cm2l2YgnrXwOaGkSd4sBBxAAAAABJRU5ErkJggg==')no-repeat 50% 50%; */
  /* 原项目的图标 */
  /* background: url('data:image/svg+xml;base64,PHN2ZyBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB3aWR0aD0iMjAiIGhlaWdodD0iMjAiPjxwYXRoIGQ9Ik01NjQgMTgwLjJINDQ4Yy04LjMgMC0xNS02LjctMTUtMTVzNi43LTE1IDE1LTE1aDExNmM4LjIgMCAxNSA2LjcgMTUgMTVzLTYuOCAxNS0xNSAxNXoiIGZpbGw9IiM5ODk4OTgiLz48cGF0aCBkPSJNOTQ1IDk1Mi4ySDgxLjJjLTguMiAwLTE1LTYuNy0xNS0xNVYxNjIuOGMwLTguMyA2LjgtMTUgMTUtMTVIMjk0YzguMiAwIDE1IDYuNyAxNSAxNXMtNi44IDE1LTE1IDE1SDk2LjJ2NzQ0LjRIOTMwVjE3Ny44SDcxMy42Yy04LjMgMC0xNS02LjctMTUtMTVzNi43LTE1IDE1LTE1SDk0NWM4LjIgMCAxNSA2LjcgMTUgMTV2Nzc0LjRjMCA4LjMtNi44IDE1LTE1IDE1eiIgZmlsbD0iIzk4OTg5OCIvPjxwYXRoIGQ9Ik0zMzMuMyA1NTFIMjE2Yy04LjIgMC0xNS02LjgtMTUtMTVzNi44LTE1IDE1LTE1aDExNy4zYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1em0yMzAuMyAwSDQ0Ni4zYy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4zYzguMiAwIDE1IDYuNiAxNSAxNXMtNi44IDE1LTE1IDE1em0yMzAuMiAwSDY3Ni42Yy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4yYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1ek0zMzMuMyA3NDBIMjE2Yy04LjIgMC0xNS02LjgtMTUtMTVzNi44LTE1IDE1LTE1aDExNy4zYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1em0yMzAuMyAwSDQ0Ni4zYy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4zYzguMiAwIDE1IDYuNiAxNSAxNXMtNi44IDE1LTE1IDE1em0yMzAuMiAwSDY3Ni42Yy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4yYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1ek0zNzAuOCAyNTguNmMtOC4zIDAtMTUtNi43LTE1LTE1Vjg2LjhjMC04LjIgNi43LTE1IDE1LTE1czE1IDYuOCAxNSAxNXYxNTYuOGMwIDguMy02LjcgMTUtMTUgMTV6bTI3MC4yIDBjLTguMyAwLTE1LTYuNy0xNS0xNVY4Ni44YzAtOC4yIDYuNy0xNSAxNS0xNXMxNSA2LjggMTUgMTV2MTU2LjhjMCA4LjMtNi43IDE1LTE1IDE1ek05NDUgMzcyLjJIODEuMmMtOC4yIDAtMTUtNi43LTE1LTE1czYuOC0xNSAxNS0xNUg5NDVjOC4yIDAgMTUgNi43IDE1IDE1cy02LjggMTUtMTUgMTV6IiBmaWxsPSIjOTg5ODk4Ii8+PC9zdmc+') no-repeat 50% 50%; */
  background-size: 20px 20px;
}

@media (max-width: 360px) {
  .datepicker:before {
    width: 34px;
    height: 34px;
    left: 8%;
  }
}

.datepicker-close {
  display: none;
  position: absolute;
  width: 34px;
  height: 100%;
  top: 0;
  right: 0;
  cursor: pointer;
}

.datepicker-close:before {
  display: block;
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  left: 50%;
  top: 50%;
  margin-left: -8px;
  margin-top: -8px;
  text-align: center;
  background: #ccc;
  color: #fff;
  border-radius: 50%;
  background:#ccc url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA3IDciIHdpZHRoPSI3IiBoZWlnaHQ9IjciPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik01LjU4LDVsMi44LTIuODFBLjQxLjQxLDAsMSwwLDcuOCwxLjZMNSw0LjQxLDIuMiwxLjZhLjQxLjQxLDAsMCwwLS41OC41OGgwTDQuNDIsNSwxLjYyLDcuOGEuNDEuNDEsMCwwLDAsLjU4LjU4TDUsNS41OCw3LjgsOC4zOWEuNDEuNDEsMCwwLDAsLjU4LS41OGgwWiIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoLTEuNSAtMS40OCkiIHN0eWxlPSJmaWxsOiNmZmYiLz48L3N2Zz4NCg==') no-repeat 50% 50%;
}

.datepicker__clearable:hover:before {
  display: none;
}
.datepicker__clearable:hover .datepicker-close{
  display: block;
}

.datepicker-close:hover:before{
  background-color: #afafaf;
}

.datepicker>input {
  /* color: #666; */
  color: #333;
  transition: all 200ms ease;
  /* border: 1px solid #e5e5e5; */
  /* border: 1px solid #d71414; */
  border: 0px;
  -webkit-box-shadow: 0 5px 10px -5px rgb(180, 180, 180, .5);
  box-shadow: 0 5px 10px -5px rgb(180, 180, 180, .5);
  border-radius: 30px;
  height: 40px;
  text-align: center;
  box-sizing: border-box;
  outline: none;
  padding: 0 4px 0 16px;
  /* padding: 0 34px 0 12px; */
  font-size: 16px;
  width: 100%;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
}

.datepicker>input.focus {
  border-color: #3bb4f2;
  -webkit-box-shadow: 0 0 5px rgba(59, 180, 242, .3);
  box-shadow: 0 0 5px rgba(59, 180, 242, .3);
}

.datepicker>input:disabled {
  cursor: not-allowed;
  background-color: #ebebe4;
  border-color: #e5e5e5;
  -webkit-box-shadow: none;
  box-shadow: none;
}

.datepicker-popup {
  position: absolute;
  transition: all 200ms ease;
  opacity: 1;
  transform: scaleY(1);
  transform-origin: center top;
  font-size: 12px;
  background: #fff;
  border: 1px solid #d9d9d9;
  box-shadow: 0 1px 6px rgba(99, 99, 99, 0.2);
  margin-top: 2px;
  outline: 0;
  padding: 5px;
  overflow: hidden;
  z-index: 999
}

.datepicker-inline{
  position: relative;
  margin-top: 0;
}

.datepicker-range {
  min-width: 325px
}

.datepicker-range .datepicker-popup{
  width: 403px;
}

.datepicker-bottom {
  float: left;
  width: 100%;
  text-align: right;
}

.datepicker-btn {
  padding: 5px 10px;
  background: #1284e7;
  color: #fff;
  border-radius: 2px;
  display: inline-block;
  cursor: pointer;
}
.datepicker-anim-enter-active {
    transform-origin: 0 0;
    animation: datepicker-anim-in .2s cubic-bezier(.23, 1, .32, 1)
}

.datepicker-anim-leave-active {
    transform-origin: 0 0;
    animation: datepicker-anim-out .2s cubic-bezier(.755, .05, .855, .06)
}

.datepicker__buttons {
  display: block;
  text-align: right;
}

.datepicker__buttons button {
  display: inline-block;
  font-size: 13px;
  border: none;
  cursor: pointer;
  margin: 10px 0 0 5px;
  padding: 5px 15px;
  color: #ffffff;
}

.datepicker__buttons .datepicker__button-select {
  background: #1284e7;
}

.datepicker__buttons .datepicker__button-cancel {
  background: #666;
}

@keyframes datepicker-anim-in {
    0% {
        opacity: 0;
        transform: scaleY(.8)
    }
    to {
        opacity: 1;
        transform: scaleY(1)
    }
}

@keyframes datepicker-anim-out {
    0% {
        opacity: 1;
        transform: scaleY(1)
    }
    to {
        opacity: 0;
        transform: scaleY(.8)
    }
}
</style>
