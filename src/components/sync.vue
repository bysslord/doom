<template>
  <div :class="templateClass">
    <div class="columns">
      <div class="column is-8 is-offset-2">
        <div class="content">
          <strong>提示:</strong>
          <ul>同步之前请先保存, 否则将同步上一次的配置</ul>
          <p class="control">
            <a class="button is-primary" @click="sync">同步</a>
            <a class="button is-danger" @click="stop">停止</a>
            <a class="button is-warning" @click="clear">清空</a>
          </p>
          <article class="message is-primary">
            <div class="message-header">
              同步日志
            </div>
            <div class="message-body">
              <div class="log" id="log">
                <li v-for="log in logs" track-by="$index">{{ log }}</li>
              </div>
            </div>
          </article>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import CONST from '../const'

  export default{
    data () {
      return {
        templateClass: {
          'is-hidden': true
        },
        logs: [],
        timer: null
      }
    },
    methods: {
      alert: function (body) {
        this.$parent.alert(body)
      },
      show: function () {
        this.templateClass['is-hidden'] = false
      },
      hidden: function () {
        this.templateClass['is-hidden'] = true
      },
      sync: function () {
        if (this.$parent.state !== CONST.state.idle) {
          this.alert('当前正在' + this.$parent.state + ', 请稍后再试')
          return
        }
        if (this.timer) {
          this.alert('已在运行')
        } else {
          this.$http.get('/api/run').then(function (response) {
            this.timer = setTimeout(this.log, 300)
            this.$parent.state = CONST.state.syncing
          }, function (response) {
            this.alert(response.data)
          })
        }
      },
      stop: function () {
        this.$http.get('/api/stop').then(function (response) {
          if (this.timer) {
            clearTimeout(this.timer)
            this.timer = null
            this.$parent.state = CONST.state.idle
          } else {
            this.alert('没有在运行')
          }
        }, function (response) {
          this.alert(response.data)
        })
      },
      clear: function () {
        this.logs = []
      },
      log: function () {
        var isFinished = false
        this.$http.get('/api/log').then(function (response) {
          for (var i in response.data) {
            this.logs.push(response.data[i])
            if (response.data[i] === 'task finished') {
              clearTimeout(this.timer)
              this.timer = null
              isFinished = true
              this.$parent.state = CONST.state.idle
            }
          }
        })
        var log = document.getElementById('log')
        log.scrollTop = log.scrollHeight
        if (!isFinished) {
          this.timer = setTimeout(this.log, 300)
        }
      }
    }
  }
</script>

<style>
  @import "../assets/css/bulma.css";
  .log {
    height: 600px;
    overflow-y: scroll;
    overflow-x: scroll;
    font-size: small;
  }
</style>
