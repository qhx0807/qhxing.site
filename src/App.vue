<template>
  <div id="app">
    <keep-alive>
      <router-view/>
    </keep-alive>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
export default {
  name: 'app',
  data() {
    return {
      hearts: [],
    }
  },
  created() {
    this.attachEvent()
    this.loop()
    this.getUserIp()
  },
  mounted() {
    this.scrollWatch()
  },
  methods: {
    randomColor() {
      return (
        'rgb(' +
        ~~(Math.random() * 255) +
        ',' +
        ~~(Math.random() * 255) +
        ',' +
        ~~(Math.random() * 255) +
        ')'
      )
    },
    attachEvent() {
      let t = this
      var old = typeof window.onclick === 'function' && window.onclick
      window.onclick = function(event) {
        old && old()
        t.createHeart(event)
      }
    },
    createHeart(event) {
      var d = document.createElement('div')
      d.className = 'heart'
      this.hearts.push({
        el: d,
        x: event.clientX - 5,
        y: event.clientY - 5,
        scale: 1,
        alpha: 1,
        color: this.randomColor()
      })
      document.body.appendChild(d)
    },
    loop() {
      for (let i = 0; i < this.hearts.length; i++) {
        if (this.hearts[i].alpha <= 0) {
          document.body.removeChild(this.hearts[i].el)
          this.hearts.splice(i, 1)
          continue
        }
        this.hearts[i].y--
        this.hearts[i].scale += 0.008
        this.hearts[i].alpha -= 0.013
        this.hearts[i].el.style.cssText =
          'left:' +
          this.hearts[i].x +
          'px;top:' +
          this.hearts[i].y +
          'px;opacity:' +
          this.hearts[i].alpha +
          ';transform:scale(' +
          this.hearts[i].scale +
          ',' +
          this.hearts[i].scale +
          ') rotate(45deg);background:' +
          this.hearts[i].color
      }
      let reqAniFrame =
        window.requestAnimationFrame ||
        window.webkitRequestAnimationFrame ||
        window.mozRequestAnimationFrame ||
        window.oRequestAnimationFrame ||
        window.msRequestAnimationFrame ||
        function(callback) {
          setTimeout(callback, 1000 / 60)
        }

      reqAniFrame(this.loop)
    },
    scrollWatch(fn){
      let t = this
      document.addEventListener('mousewheel', function(e){
        if(e.wheelDelta < 0){
          t.$store.commit('UPDATE_HEADER', -102)
        }else{
          t.$store.commit('UPDATE_HEADER', 0)
        }
      }, false)
    },
    getUserIp(){
      axios.get('http://localhost:3000/ip').then(function(response){
        
        let str = response.data.Data.split(",")
        let ip = str[0].split(':')[1].replace(/\'/g, '')
        let addr = str[1].split(':')[1].replace(/\'/g, '').replace('}', '')
        // let d = JSON.parse(str)
        sessionStorage.setItem('ip', ip)
        sessionStorage.setItem('city', addr)
        
        this.addPvRecord(ip, addr)
      }.bind(this)).catch(function(error){
        console.log(error)
      }.bind(this))
    },
    addPvRecord(ip, addr){
      let pvData = {
        time: moment().format().replace('T', ' ').substring(0,19),
        ip: ip,
        location: addr,
      }
      axios.post(API_URL + '/pv', pvData).then(function(response) {
        if (response.data.OK) {
        }
      }.bind(this)).catch(function(error) {
      }.bind(this))
    }
  }
}
</script>

<style leng="less">
@import url('//at.alicdn.com/t/font_473583_52w68mpoa0n6d2t9.css');
html,
body {
  padding: 0;
  margin: 0;
  height: 100%;
  width: 100%;
}
@font-face {
    font-family: "qhx";
    src: url('./assets/qhx.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}

::-webkit-scrollbar {
    width: 8px;
    height: 8px;
    background-color: #B9B9B9;
}
::-webkit-scrollbar-thumb {
    border-radius: 10px;
    background-color: #A0DAD0;
}
::selection {
    background: #A0DAD0 !important;
    color: #fff;
}
#app {
  font-family: "qhx","Helvetica Neue","Helvetica","Arial","PingFang SC","Hiragino Sans GB", "Microsoft YaHei","WenQuanYi Micro Hei", sans-serif !important;
  -webkit-text-size-adjust: 100%;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 16px !important;
  line-height: 24px !important;
  height: 100%;
  width: 100%;
  /* background-color: #f8f8f8; */
}
.heart {
  width: 8px;
  height: 8px;
  position: fixed;
  background: deepskyblue;
  transform: rotate(45deg);
  z-index: 100;
}
.heart:after,
.heart:before {
  content: '';
  width: inherit;
  height: inherit;
  background: inherit;
  border-radius: 50%;
  position: absolute;
}
.heart:after {
  top: -5px;
}
.heart:before {
  left: -5px;
}
</style>
