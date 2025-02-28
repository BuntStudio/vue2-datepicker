<template>
  <div :class="`${prefixClass}-calendar ${prefixClass}-calendar-panel-month`">
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
        <button
          type="button"
          :class="`${prefixClass}-btn ${prefixClass}-btn-text`"
          @click="handlePanelChange"
        >
          {{ calendarYear }}
        </button>
      </span>
    </div>
    <div :class="`${prefixClass}-calendar-content`">
      <table :class="`${prefixClass}-table ${prefixClass}-table-month`" @click="handleClick">
        <tr v-for="(row, i) in months" :key="i">
          <template v-for="(cell, j) in row">
            <!--            v-if="getCellClasses(cell.month).indexOf('disabled') === -1"-->
            <td :key="j" :data-month="cell.month" class="cell" :class="getCellClasses(cell.month)">
              <div>{{ cell.text }}</div>
            </td>
          </template>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import { chunk } from '../util/base';
import IconButton from './icon-button';
import { getLocale } from '../locale';
import { setYear } from '../util/date';

export default {
  name: 'TableMonth',
  components: { IconButton },
  inject: {
    getLocale: {
      default: () => getLocale,
    },
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
    minDate: {
      type: Function,
    },
    maxDate: {
      type: Function,
    },
    disableYear: {
      type: Boolean,
    },
  },
  computed: {
    calendarYear() {
      return this.calendar.getFullYear();
    },
    months() {
      const locale = this.getLocale();
      const monthsLocale = locale.months || locale.formatLocale.monthsShort;
      const months = monthsLocale
        .map((text, month) => {
          return { text, month };
        })
        .filter((text, month) => this.getCellClasses(month).indexOf('disabled') === -1);
      return chunk(months, 3);
    },
    disableLeftClick() {
      return this.checkIfYearNotAvailable(this.calendarYear - 1);
      // return (
      //   typeof this.minDate === 'function' &&
      //   this.minDate() &&
      //   this.minDate().getFullYear() > this.calendarYear - 1
      // );
    },
    disableRightClick() {
      return this.checkIfYearNotAvailable(this.calendarYear + 1);
      // return (
      //   typeof this.maxDate === 'function' &&
      //   this.maxDate() &&
      //   this.maxDate().getFullYear() < this.calendarYear + 1
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
    handleIconDoubleLeftClick() {
      const yearsToDecrease = this.disableLeftClick ? 0 : 1;

      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v - yearsToDecrease),
        'last-year'
      );
    },
    handleIconDoubleRightClick() {
      const yearsToIncrease = this.disableRightClick ? 0 : 1;
      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v + yearsToIncrease),
        'next-year'
      );
    },
    handlePanelChange() {
      this.$emit('changepanel', 'year');
    },
    handleClick(evt) {
      let { target } = evt;
      if (target.tagName.toUpperCase() === 'DIV') {
        target = target.parentNode;
      }
      const month = target.getAttribute('data-month');
      const monthDisabled = target.getAttribute('class').indexOf('disabled') > -1;
      if (month && !monthDisabled) {
        this.$emit('select', parseInt(month, 10));
      }
    },
  },
};
</script>
