<template>
  <div id="wrapper">
      <div class="bg"></div>
      <div class="bgImage" v-on:keyup="keymonitor"></div>

      <div class="progress" style="height:10px">
        <div class="progress-bar progress-bar-striped bg-info progress-bar-animated" role="progressbar" aria-valuenow="75" aria-valuemin="0" aria-valuemax="100" :style="'width:' + percentage + '%'"></div>
      </div>

      <!-- <p :style="'position: absolute; font-size: 2em; right: calc(' + (100 - percentage) + '% + 5px); top: 0px'">Sep 19, 2018</p>
      <p :style="'position: absolute; font-size: 2em; right: calc(' + (100 - percentage) + '% + 5px); top: 70px'">15:23:37</p> -->

      <p v-for="(element, index) in hudElements" v-bind:key="element" v-bind:style="'position: absolute; font-size: 2em; right: 10px; top: ' + 35 * index + 'px; color:white'">{{element}}</p>
      <p v-bind:style="'position: absolute; font-size: 2em; right: 10px; bottom: 0px; color:white'">{{keyPress}}</p>

      <div v-if="viewTasks" id="tasks" class="tasks">
        <div style="display: block-inline"><h5>Tasks</h5></div>
        <hr style="border-color: rgba(255,255,255,0.5); margin-top: 2px">
        <table style="width: 100%">
          <tr>
            <th width="30px"></th>
            <th style="width: 60px">Task ID</th>
            <th>Task Description</th>
          </tr>
          <tr v-for="(task, index) in tasks.data">
            <td style="-webkit-app-region: no-drag; padding-bottom: 5px"><button class="btn btn-sm btn-danger" @click="deleteTask(index)">X</button></td>
            <td style="text-align: center">{{index}}</td>
            <td>{{task}}</td>
          </tr>
        </table>
      </div>

      <div v-if="viewAdd" id="addLine" class="addLine">
        <div class="form" style="-webkit-app-region: no-drag;">
          What do you want to add?
          <select class="form-group form-control">
            <option>Task</option>
            <!-- <option>Note</option> -->
          </select>
          <input v-model="newTask" @focus="setAddFocus(true)" @blur="setAddFocus(false)" class="form-group form-control" style="resize: none;"></input>
          <button @click="submitAddition()" class="btn btn-info form-group form-control">Add</button>
        </div>
        
      </div>

      <!-- <h1 :style="'position: absolute; font-size: ' + 20 * (percentage / 100) + 'em; right: calc(' + (100 - percentage) + '% + 10px); top: calc(' + 155 * (percentage / 100) + 'px)'">{{percentage}}%</h1> -->

  </div>
</template>

<script>
  var moment = require('moment')
  export default {
    name: 'landing-page',
    data () {
      return {
        addFocus: false,
        hudElements: [],
        percentage: 0,
        overtimePercent: 0,
        i: 0,
        t: '',
        d: '',
        endHour: 17,
        endMinute: 0,
        keyPress: null,
        viewEnd: false,
        viewTasks: true,
        viewAdd: false,
        newTask: '',
        tasks: {
          latest: 0,
          data: {
            0: 'This is an example task, you can create more below and delete existing ones using the button on the left.'
          }}
      }
    },
    watch: {
      endMinute: function (val) {
        if (val >= 60) {
          this.endMinute = 0
          this.endHour++
        } else if (val < 0) {
          this.endMinute = 60 + val
          this.endHour--
        }
      }
    },
    methods: {
      deleteTask (id) {
        delete this.tasks.data[id]
        window.localStorage.setItem('tasks', JSON.stringify(this.tasks))
      },
      submitAddition () {
        if (this.newTask.length > 0) {
          var taskId = this.tasks.latest + 1
          var taskDesc = this.newTask
          this.tasks.data[taskId] = taskDesc
          this.tasks.latest = taskId
          this.newTask = ''

          window.localStorage.setItem('tasks', JSON.stringify(this.tasks))
        }
      },
      setAddFocus: function (focus) {
        this.addFocus = focus
      },
      time () {
        var d = new Date()
        this.t = d.toLocaleTimeString()
        this.d = d.toDateString()

        var now = moment()
        // this.now = now
        var five = moment().hour(this.endHour).minute(this.endMinute).second(0)

        var dif = five.diff(now, 'seconds')

        var percentage = (1 - (dif / 28800)) * 100

        this.overtimePercent = 0

        if (percentage > 100) {
          this.overtimePercent = percentage - 100
          percentage = 100
        } else if (percentage < 0) {
          percentage = 0
        }

        this.percentage = percentage

        this.hudElements = [
          this.d,
          this.t,
          ((this.viewEnd) ? this.endHour + ':' + ((this.endMinute < 10) ? '0' + this.endMinute : this.endMinute) + ':00' : ((this.overtimePercent > 0) ? Math.round(this.percentage) + '% (+' + Math.round(this.overtimePercent) + '%)' : Math.round(this.percentage) + '%')),
          ((this.viewEnd) ? ((this.overtimePercent > 0) ? Math.round(this.percentage) + '% (+' + Math.round(this.overtimePercent) + '%)' : Math.round(this.percentage) + '%') : '')]

        // console.log(this.percentage)
        // console.log(five.format())
      },
      keymonitor (event) {
        this.keyPress = event.keyCode
        if (!this.addFocus) {
          switch (event.keyCode) {
            case 33:
              this.endHour++
              break
            case 34:
              this.endHour--
              break
            case 36:
              this.endMinute += 10
              break
            case 35:
              this.endMinute -= 10
              break
            case 223:
              this.viewEnd = (!this.viewEnd)
              break
            case 84:
              this.viewTasks = (!this.viewTasks)
              break
            case 112:
              this.viewAdd = (!this.viewAdd)
              break
          }
        }
      }
    },
    mounted () {
      let self = this
      window.setTimeout(function () {
        window.setInterval(function () {
          self.time()
        })
      }, 100)
      window.addEventListener('keyup', this.keymonitor)
      if (JSON.parse(window.localStorage.getItem('tasks'))) {
        this.tasks = JSON.parse(window.localStorage.getItem('tasks'))
      }
    }
  }
</script>
<style scoped>
  body {
    overflow: hidden;
  }

  .tasks {
    position: absolute;
    left: 75px;
    top: 75px;
    background-color: rgba(255,255,255,0.25);
    padding: 10px;
    border-radius: 10px;
    color: white;
    width: 500px;
  }

  .addLine {
    position: absolute;
    left: 75px;
    bottom: 20px;
    background-color: rgba(255,255,255,0.25);
    padding: 10px;
    border-radius: 10px;
    color: #eee;
    font-weight: bold;
    width: calc(100vw - 150px);
  }

  .tasks ul, .tasks li {
    list-style: none;
  }

  .bg {
    position: absolute;
    width: 100vw;
    height: 100vh;
    z-index: -2;
    background-color: black;
    background-size: cover;
    background-position: center;
  }
  .bgImage {
    position: absolute;
    width: 100vw;
    height: 100vh;
    z-index: -1;
    background-image: url('../assets/bg.jpg');
    background-size: cover;
    background-position: center;
    filter: blur(5px);
    -webkit-app-region: drag
  }

  .progress {
    background-color: rgba(0,0,0,0);
    z-index: 1;
  }

  .progress-bar {
    border-right: 5px dashed black;
    border-radius: 0;
    opacity: 0.75;
    z-index: 2;
  }

  h1, p {
    z-index: 3;
    color: black;
  }

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

</style>
