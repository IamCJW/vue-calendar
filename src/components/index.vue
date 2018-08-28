<template>
  <div>
    <calendar :prop_call_back="callBack"></calendar>
    <div style="font-size:14px">您选择了{{year}}-{{month}}-{{day}}</div>
  </div>
</template>

<script>
  import calendar from './calendar'
  export default {
    name: "index",
    data(){
      return{
        year:"",
        month:"",
        day:"",
      }
    },
    components: {calendar:calendar},
    methods:{
      callBack(date){
        this.year = date.year;
        this.month = date.month;
        this.day = date.day;
      }
    }
  }
</script>

<style scoped>

</style>
