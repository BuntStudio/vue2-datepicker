<template>
  <div class="box">
    <section>
      <p>date range</p>
      <date-picker
        v-model="value1"
        type="date"
        :max-date="() => new Date(maxDate)"
        :min-date="() => new Date(minDate)"
        :confirm-text="'Apply'"
        :disabled-date="disableDatesByMinAndMaxDate"
        :range-separator="' - '"
        range
        placeholder="Select date range"
      ></date-picker>
    </section>
    <section>
      <p>datetime range</p>
      <date-picker
        v-model="value2"
        :max-date="() => new Date(maxDate)"
        :min-date="() => new Date(minDate)"
        type="datetime"
        :disabled-date="disableDatesByMinAndMaxDate"
        range
        placeholder="Select datetime range"
      ></date-picker>
    </section>
  </div>
</template>

<script>
export default {
  name: 'Range',
  data() {
    return {
      value1: [new Date(2022, 2, 14, 2), new Date(2022, 3, 14, 3)],
      value2: [],
      maxDate: '2022-04-14',
      minDate: '2021-08-19',
      // ["[native Date Mon Mar 14 2022 02:00:00 GMT+0200 (Eastern European Standard Time)]","[native Date Thu Apr 14 2022 03:00:00 GMT+0300 (Eastern European Summer Time)]"]
    };
  },
  methods: {
    disableDatesByMinAndMaxDate(date) {
      // const minDateObj = new Date(this.minCalendarDate);
      const minDateObj = new Date(this.minDate);
      const maxDateObj = new Date(this.maxDate);

      // let compareActive = false;

      minDateObj.setHours(0, 0, 0, 0);
      maxDateObj.setHours(0, 0, 0, 0);

      // if(this.compareStartDate && this.compareEndDate){
      //     const compareStartDateObj = new Date(this.compareStartDate);
      //     const compareEndDateObj = new Date(this.compareEndDate);
      //
      //     compareStartDateObj.setHours(0, 0, 0, 0);
      //     compareEndDateObj.setHours(0, 0, 0, 0);
      //
      //     compareActive = date >= compareStartDateObj && date <= compareEndDateObj;
      // }

      return date < minDateObj || date > maxDateObj;
    },
  },
};
</script>
