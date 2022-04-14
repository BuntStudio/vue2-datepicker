<template>
  <div :class="`${prefixClass}-calendar ${prefixClass}-calendar-panel-date`">
    <div :class="`${prefixClass}-calendar-header`">
      <icon-button
        :class="{ disabled: disableDoubleLeftClick }"
        type="double-left"
        @click="handleIconDoubleLeftClick"
      ></icon-button>
      <icon-button
        :class="{ disabled: disableLeftClick }"
        type="left"
        @click="handleIconLeftClick"
      ></icon-button>
      <icon-button
        :class="{ disabled: disableDoubleRightClick }"
        type="double-right"
        @click="handleIconDoubleRightClick"
      ></icon-button>
      <icon-button
        :class="{ disabled: disableRightClick }"
        type="right"
        @click="handleIconRightClick"
      ></icon-button>
      <span :class="`${prefixClass}-calendar-header-label`">
        <button
          v-for="item in yearMonth"
          :key="item.panel"
          type="button"
          :class="
            `${prefixClass}-btn ${prefixClass}-btn-text ${prefixClass}-btn-current-${item.panel}`
          "
          @click="handlePanelChange(item.panel)"
        >
          {{ item.label }}
        </button>
      </span>
    </div>
    <div :class="`${prefixClass}-calendar-content`">
      <table :class="`${prefixClass}-table ${prefixClass}-table-date`">
        <thead>
          <tr>
            <th v-if="showWeekNumber" :class="`${prefixClass}-week-number-header`"></th>
            <th v-for="day in days" :key="day">{{ day }}</th>
          </tr>
        </thead>
        <tbody @click="handleCellClick">
          <tr
            v-for="(row, i) in dates"
            :key="i"
            :class="[`${prefixClass}-date-row`, getRowClasses(row)]"
          >
            <td
              v-if="showWeekNumber"
              :data-row-col="`${i},0`"
              :class="`${prefixClass}-week-number`"
            >
              {{ getWeekNumber(row[0]) }}
            </td>
            <td
              v-for="(cell, j) in row"
              :key="j"
              :data-row-col="`${i},${j}`"
              class="cell"
              :class="getCellClasses(cell)"
              :title="getCellTitle(cell)"
              @mouseenter="handleMouseEnter(cell)"
              @mouseleave="handleMouseLeave(cell)"
            >
              <div>{{ cell.getDate() }}</div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { getWeek, format } from 'date-format-parse';
import IconButton from './icon-button';
import { chunk } from '../util/base';
import { getCalendar, setMonth, setYear } from '../util/date';
import { getLocale } from '../locale';

export default {
  name: 'TableDate',
  components: { IconButton },
  inject: {
    getLocale: {
      default: () => getLocale,
    },
    getWeek: {
      default: () => getWeek,
    },
    prefixClass: {
      default: 'mx',
    },
    onDateMouseEnter: {
      default: undefined,
    },
    onDateMouseLeave: {
      default: undefined,
    },
  },
  props: {
    calendar: {
      type: Date,
      default: () => new Date(),
    },
    showWeekNumber: {
      type: Boolean,
      default: false,
    },
    titleFormat: {
      type: String,
      default: 'YYYY-MM-DD',
    },
    getRowClasses: {
      type: Function,
      default: () => [],
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
  },
  computed: {
    firstDayOfWeek() {
      return this.getLocale().formatLocale.firstDayOfWeek || 0;
    },
    yearMonth() {
      const { yearFormat, monthBeforeYear, monthFormat = 'MMM' } = this.getLocale();
      const yearLabel = {
        panel: 'year',
        label: this.formatDate(this.calendar, yearFormat),
      };
      const monthLabel = {
        panel: 'month',
        label: this.formatDate(this.calendar, monthFormat),
      };
      return monthBeforeYear ? [monthLabel, yearLabel] : [yearLabel, monthLabel];
    },
    days() {
      const locale = this.getLocale();
      const days = locale.days || locale.formatLocale.weekdaysMin;
      return days.concat(days).slice(this.firstDayOfWeek, this.firstDayOfWeek + 7);
    },
    dates() {
      const year = this.calendar.getFullYear();
      const month = this.calendar.getMonth();
      const arr = getCalendar({
        firstDayOfWeek: this.firstDayOfWeek,
        year,
        month,
      });
      return chunk(arr, 7);
    },
    disableLeftClick() {
      return this.disableDoubleLeftClick && this.minDate().getMonth() >= this.calendar.getMonth();
    },
    disableRightClick() {
      return this.disableDoubleRightClick && this.maxDate().getMonth() <= this.calendar.getMonth();
    },
    disableDoubleLeftClick() {
      return (
        typeof this.minDate === 'function' &&
        this.minDate() &&
        this.minDate().getFullYear() > this.calendar.getFullYear() - 1
      );
    },
    disableDoubleRightClick() {
      return (
        typeof this.maxDate === 'function' &&
        this.maxDate() &&
        this.maxDate().getFullYear() < this.calendar.getFullYear() + 1
      );
    },
  },
  methods: {
    handleIconLeftClick() {
      const monthToDecrease = this.disableLeftClick ? 0 : 1;

      this.$emit(
        'changecalendar',
        setMonth(this.calendar, v => v - monthToDecrease),
        'last-month'
      );
    },
    handleIconRightClick() {
      const monthToIncrease = this.disableRightClick ? 0 : 1;
      this.$emit(
        'changecalendar',
        setMonth(this.calendar, v => v + monthToIncrease),
        'next-month'
      );
    },
    handleIconDoubleLeftClick() {
      const yearsToDecrease = this.disableDoubleLeftClick ? 0 : 1;

      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v - yearsToDecrease),
        'last-year'
      );
    },
    handleIconDoubleRightClick() {
      const yearsToIncrease = this.disableDoubleRightClick ? 0 : 1;

      this.$emit(
        'changecalendar',
        setYear(this.calendar, v => v + yearsToIncrease),
        'next-year'
      );
    },
    handlePanelChange(panel) {
      this.$emit('changepanel', panel);
    },
    handleMouseEnter(cell) {
      if (typeof this.onDateMouseEnter === 'function') {
        this.onDateMouseEnter(cell);
      }
    },
    handleMouseLeave(cell) {
      if (typeof this.onDateMouseLeave === 'function') {
        this.onDateMouseLeave(cell);
      }
    },
    handleCellClick(evt) {
      let { target } = evt;
      if (target.tagName.toUpperCase() === 'DIV') {
        target = target.parentNode;
      }
      const index = target.getAttribute('data-row-col');
      if (index) {
        const [row, col] = index.split(',').map(v => parseInt(v, 10));
        const date = this.dates[row][col];
        this.$emit('select', new Date(date));
      }
    },
    formatDate(date, fmt) {
      return format(date, fmt, { locale: this.getLocale().formatLocale });
    },
    getCellTitle(date) {
      const fmt = this.titleFormat;
      return this.formatDate(date, fmt);
    },
    getWeekNumber(date) {
      return this.getWeek(date, this.getLocale().formatLocale);
    },
  },
};
</script>
