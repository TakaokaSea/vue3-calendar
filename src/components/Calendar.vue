<template>
  <div class="content">
    <h2>カレンダー{{ displayMonth }}</h2>
    <div class="button-area">
      <button @click="prevMonth" class="button">前の月</button>
      <button @click="nextMonth" class="button">次の月</button>
    </div>
    <div class="calendar">
      <div class="calendar-weekly">
        <div class="calendar-youbi" v-for="n in 7" :key="n">
          {{ youbi(n-1) }}
        </div>
      </div>
      <div 
        class="calendar-weekly"
        v-for="(week, index) in calendars"
        :key="index"
      >
        <div
          class="calendar-daily"
          :class="{outside: currentMonth !== day.month}"
          v-for="(day, index) in week"
          :key="index"
          @drop="dragEnd"
          @dragover.prevent
        >
          <div class="calendar-day">
            {{ day.day }}
          </div>
          <div v-for="dayEvent in day.dayEvents" :key="dayEvent.id" >
            <div
              v-if="dayEvent.width"
              class="calendar-event"
              :style="`width:${dayEvent.width}%;background-color:${dayEvent.color}`" 
              draggable="true"
              @dragstart="dragStart(dayEvent.id)"
              @drop="dragEnd(day.date)">
              {{ dayEvent.name }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
export default {
  data() {
    return {
      currentDate: moment(),
      events:[
        { id: 1, name: "ミーティング", start: "2021-02-01", end:"2021-02-01", color:"blue"},
        { id: 2, name: "イベント", start: "2021-02-02", end:"2021-02-03", color:"limegreen"},
        { id: 3, name: "会議", start: "2021-02-06", end:"2021-02-06", color:"deepskyblue"},
        { id: 4, name: "有休", start: "2021-02-08", end:"2021-02-08", color:"dimgray"},
        { id: 5, name: "海外旅行", start: "2021-02-08", end:"2021-02-11", color:"navy"},
        { id: 6, name: "誕生日", start: "2021-02-16", end:"2021-02-16", color:"orange"},
        { id: 7, name: "イベント", start: "2021-02-12", end:"2021-02-15", color:"limegreen"},
        { id: 8, name: "出張", start: "2021-02-12", end:"2021-02-13", color:"teal"},
        { id: 9, name: "客先訪問", start: "2021-02-14", end:"2021-02-14", color:"red"},
        { id: 10, name: "パーティ", start: "2021-02-15", end:"2021-02-15", color:"royalblue"},
        { id: 12, name: "ミーティング", start: "2021-02-18", end:"2021-02-19", color:"blue"},
        { id: 13, name: "イベント", start: "2021-02-21", end:"2021-02-21", color:"limegreen"},
        { id: 14, name: "有休", start: "2021-02-20", end:"2021-02-20", color:"dimgray"},
        { id: 15, name: "イベント", start: "2021-02-25", end:"2021-02-28", color:"limegreen"},
        { id: 16, name: "会議", start: "2021-02-21", end:"2021-02-21", color:"deepskyblue"},
        { id: 17, name: "旅行", start: "2021-02-23", end:"2021-02-24", color:"navy"},
        { id: 18, name: "ミーティング", start: "2021-02-28", end:"2021-02-28", color:"blue"},
        { id: 19, name: "会議", start: "2021-02-12", end:"2021-02-12", color:"deepskyblue"},
        { id: 20, name: "誕生日", start: "2021-02-30", end:"2021-02-30", color:"orange"},
      ],
    }
  },
  
  methods: {
    getStartDate() {
      let date = moment(this.currentDate);
      date.startOf("month");
      const youbiNum = date.day();
      return date.subtract(youbiNum, "days");
    },
    getEndDate() {
      let date = moment(this.currentDate);
      date.endOf("month");
      const youbiNum = date.day();
      return date.add(6 - youbiNum, "days");
    },
    getCalendar() {
      let startDate = this.getStartDate();
      const endDate = this.getEndDate();
      const weekNumber = Math.ceil(endDate.diff(startDate, "days") / 7);

      let calendars = [];
      let calendarDate = this.getStartDate();
        for (let week = 0; week < weekNumber; week++) {
          let weekRow = [];
          for (let day = 0;  day < 7; day++) {
            let dayEvents = this.getDayEvents(calendarDate, day)
            weekRow.push({
              day: calendarDate.get("date"),
              month: calendarDate.format("YYYY-MM"),
              date: calendarDate.format("YYYY-MM-DD"),
              dayEvents:dayEvents,
            });
            calendarDate.add(1, "days");
          }
          calendars.push(weekRow);
        }
      return calendars;
    },
    nextMonth() {
      this.currentDate = moment(this.currentDate).add(1, "month");
    },
    prevMonth() {
      this.currentDate = moment(this.currentDate).subtract(1, "month");
    },
    youbi(dayIndex) {
      const week = ["日", "月", "火", "水", "木", "金", "土"];
      return week[dayIndex];
    },
    getDayEvents(date, day){
      let stackIndex = 0;
      let dayEvents = [];
      let startedEvents = [];
      this.sortedEvents.forEach(event => {
        let startDate = moment(event.start).format('YYYY-MM-DD')
        let endDate = moment(event.end).format('YYYY-MM-DD')
        let Date = date.format('YYYY-MM-DD')

        // if(startDate <= Date && endDate >= Date){
        //       if(startDate === Date){
        //         [stackIndex, dayEvents, startedEvents] = this.getStackEvents(event, day, date, stackIndex, dayEvents, startedEvents, event.start);
        //       }else if(day === 0){
        //         [stackIndex, dayEvents, startedEvents] = this.getStackEvents(event, day, date, stackIndex, dayEvents, startedEvents, date);
        //       }else{
        //         startedEvents.push(event)
        //       }
        //     }
        //   });
        //   return dayEvents;
        // },


        if(startDate <= Date && endDate >= Date){
          if(startDate === Date){
            let width = this.getEventWidth(startDate, endDate, day)
            dayEvents.push({...event,width})
          }else if(day === 0){
            let width = this.getEventWidth(date, endDate, day)
            dayEvents.push({...event,width})
          }
          // else{
          // 　let width = this.getEventWidth(date, endDate, day)
          // 　dayEvents.push({...event,width})
          // }
        }
      });
      return dayEvents;
    },
    getEventWidth(start, end, day){
      let betweenDays = moment(end).diff(moment(start), "days")
      if(betweenDays > 6 - day){
        return (6 - day) * 100 + 95; 
      }else{
        return betweenDays * 100 + 95;
      }
    },

    // dragStart(eventId){
    //   console.log(eventId);
    // }
    dragStart(dayEvent){
      event.dataTransfer.effectAllowed = "move";
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.setData("eventId", dayEvent.id);
    },
    // dragEnd(){
    //   console.log(event.dataTransfer.getData("eventId"));
    //   // @drop="dragEnd(day.date)"
    // }
    // dragEnd(date){
    //   console.log(event.dataTransfer.getData("eventId"));
    //   console.log(date);
    // }
    dragEnd(date){
      let eventId = event.dataTransfer.getData("eventId");
      let dragEvent = this.events.find(event => event.id == eventId)
      let betweenDays = moment(dragEvent.end).diff(moment(dragEvent.start), "days");
      dragEvent.start = date;
      dragEvent.end = moment(dragEvent.start).add(betweenDays, "days").format("YYYY-MM-DD");
    },
  },

  computed: {
    calendars() {
      return this.getCalendar();
    },
    displayMonth(){
      return this.currentDate.format('YYYY[年]M[月]')
    },
    currentMonth(){
      return this.currentDate.format('YYYY-MM')
    },
    sortedEvents(){
      return this.events.slice().sort(function(a,b) {
        let startDate = moment(a.start).format('YYYY-MM-DD')
        let endDate = moment(b.start).format('YYYY-MM-DD')
        if( startDate < endDate ) return -1;
        if( startDate > endDate ) return 1;
        return 0;
      })
    }
  },

}
</script>

<style>
.content{
  margin:2em auto;
  width:900px;
}
.button-area{
  margin:0.5em 0;
}
.button{
  padding:4px 8px;
  margin-right:8px;
}
.calendar{
  max-width:900px;
  border-top:1px solid #E0E0E0;
  font-size:0.8em;
}
.calendar-weekly{
  display:flex;
  border-left:1px solid #E0E0E0;
  /* background-color: black; */
}
.calendar-daily{
  flex:1;
  min-height:125px;
  border-right:1px solid #E0E0E0;
  border-bottom:1px solid #E0E0E0;
  margin-right:-1px;
}
.calendar-day{
  text-align: center;
}
.calendar-youbi{
  flex:1;
  border-right:1px solid #E0E0E0;
  margin-right:-1px;
  text-align:center;
}
.outside{
  background-color: #f7f7f7;
}
.calendar-event{
  color:white;
  margin-bottom:1px;
  height:25px;
  line-height:25px;
}
.calendar-event{
  color:white;
  margin-bottom:1px;
  height:25px;
  line-height:25px;
  position: relative;
  z-index:1;
  border-radius:4px;
  padding-left:4px;
}

</style>