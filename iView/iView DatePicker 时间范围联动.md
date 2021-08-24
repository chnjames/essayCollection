<template>
  <div class="header-item">
    <div class="header-item-title">日期1</div>
    <DatePicker class="header-item-width" v-model="searchObj.schedulingTime" :options="optionScheduleRange" @on-change="scheduleTimeChange"></DatePicker>
  </div>
  <div class="header-item">
    <div class="header-item-title">日期2</div>
    <DatePicker class="header-item-width" v-model="searchObj.visitDate" :options="optionVisitRange"></DatePicker>
  </div>
</template>
<script>
export default {
  data() {
    return {
      optionScheduleRange: {  // 日期1
        disabledDate: function (date) {
          let nextDate = this.getNextDate(Date.now(), 10)
          return date && (date.valueOf() < new Date(nextDate).getTime() || date.valueOf() > Date.now());
        }.bind(this)
      },
      optionVisitRange: { // 日期2
        disabledDate: function (date) {
          let nextDate = this.getNextDate(this.searchObj.schedulingTime, -10)
          return date && (date.valueOf() > new Date(nextDate).getTime() || date.valueOf() < new Date(this.searchObj.schedulingTime).getTime());
        }.bind(this)
      },
  }, 
  mounted() {
    const start = new Date();
    this.searchObj.schedulingTime = this.dateFormat(start)
    this.searchObj.visitDate = this.dateFormat(start)
  },
  methods: {
    scheduleTimeChange(date) { /*日期1*/
      this.searchObj.visitDate = date
    },
    dateFormat(dateData) { // 标准化日期
      const date = new Date(dateData);
      const y = date.getFullYear();
      let m = date.getMonth() + 1;
      m = m < 10 ? "0" + m : m;
      let d = date.getDate();
      d = d < 10 ? "0" + d : d;
      return y + "-" + m + "-" + d;
    }
  }
}
</script>

<style lang="less" scoped>
  .header {
    display: flex;
    flex-wrap: wrap;

    &-item {
      display: flex;
      align-items: center;
      margin-right: 30px;
      margin-bottom: 10px;

      &-title {
        padding-right: 10px;
      }

      &-width {
        width: 160px;
      }
    }
  }
</style>
