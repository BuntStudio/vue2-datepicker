<template>
  <div :class="`${prefixClass}-calendar ${prefixClass}-calendar-panel-year`">
    <div :class="`${prefixClass}-calendar-header`">
      <icon-button
        :class="{ disabled: disableLeftClick }"
        type="double-left"
        @click="handleIconDoubleLeftClick"
      ></icon-button>
      <icon-button
        :class="{ disabled: disableRightClick }"
        type="double-right"
        @click="handleIconDoubleRightClick"
      ></icon-button>
      <span :class="`${prefixClass}-calendar-header-label`">
        <span>{{ firstYear }}</span>
        <span :class="`${prefixClass}-calendar-decade-separator`"></span>
        <span>{{ lastYear }}</span>
      </span>
    </div>
    <div :class="`${prefixClass}-calendar-content`">
      <table :class="`${prefixClass}-table ${prefixClass}-table-year`" @click="handleClick">
        <tr v-for="(row, i) in years" :key="i">
          <template v-for="(cell, j) in row">
            <!--            v-if="getCellClasses(cell).indexOf('disabled') === -1"-->
            <td :key="j" :data-year="cell" class="cell" :class="getCellClasses(cell)">
              <div>{{ cell }}</div>
            </td>
          </template>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import IconButton from './icon-button';
import { chunk } from '../util/base';
import { setYear } from '../util/date';

export default {
  name: 'TableYear',
  components: { IconButton },
  inject: {
    prefixClass: {
      default: 'mx',
    },
  },
  props: {
    calendar: {
      type: Date,
      default: () => new Date(),
    },
    getCellClasses: {
      type: Function,
      default: () => [],
    },
    getYearPanel: {
      type: Function,
    },
    minDate: {
      type: Function,
    },
    maxDate: {
      type: Function,
    },
  },
  computed: {
    years() {
      const calendar = new Date(this.calendar);
      if (typeof this.getYearPanel === 'function') {
        return this.getYearPanel(calendar);
      }
      return this.getYears(calendar);
    },
    firstYear() {
      return this.years[0][0];
    },
    lastYear() {
      const last = arr => arr[arr.length - 1];
      return last(last(this.years));
    },
    disableLeftClick() {
      return this.checkIfYearNotAvailable(this.lastYear - 10);
      // return (
      //   typeof this.minDate === 'function' &&
      //   this.minDate() &&
      //   this.minDate().getFullYear() > this.lastYear - 10
      // );
    },
    disableRightClick() {
      return this.checkIfYearNotAvailable(this.firstYear + 10);
      // return (
      //   typeof this.maxDate === 'function' &&
      //   this.maxDate() &&
      //   this.maxDate().getFullYear() < this.firstYear + 10
      // );
    },
  },
  methods: {
    checkIfYearNotAvailable(year) {
      return (
        (typeof this.maxDate === 'function' &&
          this.maxDate() &&
          this.maxDate().getFullYear() < year) ||
        (typeof this.minDate === 'function' &&
          this.minDate() &&
          this.minDate().getFullYear() > year)
      );
    },
    getYears(calendar) {
      const firstYear = Math.floor(calendar.getFullYear() / 10) * 10;
      const years = [];

      for (let i = 0; i < 10; i++) {
        if (!this.checkIfYearNotAvailable(firstYear + i)) {
          years.push(firstYear + i);
        }
      }

      return chunk(years, 2);
    },
    handleIconDoubleLeftClick() {
      const yearsToDecrease = this.disableLeftClick ? 0 : 10;

      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v - yearsToDecrease),
        'last-decade'
      );
    },
    handleIconDoubleRightClick() {
      const yearsToIncrease = this.disableRightClick ? 0 : 10;
      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v + yearsToIncrease),
        'next-decade'
      );
    },
    handleClick(evt) {
      let { target } = evt;
      if (target.tagName.toUpperCase() === 'DIV') {
        target = target.parentNode;
      }
      const year = target.getAttribute('data-year');
      const yearDisabled = target.getAttribute('class').indexOf('disabled') > -1;
      if (year && !yearDisabled) {
        this.$emit('select', parseInt(year, 10));
      }
    },
  },
};
</script>
