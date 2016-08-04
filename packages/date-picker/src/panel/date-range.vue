<template>
  <div
    v-show="visible"
    transition="md-fade-bottom"
    class="el-picker-panel el-date-range-picker">
    <div class="el-picker-panel__body-wrapper">
      <slot name="sidebar" class="el-picker-panel__sidebar"></slot>
      <div class="el-picker-panel__sidebar" v-if="shortcuts">
        <button
          class="el-picker-panel__shortcut"
          v-for="shortcut in shortcuts"
          @click="handleShortcutClick(shortcut)">{{shortcut.text}}</button>
      </div>
      <div class="el-picker-panel__body">
        <div class="el-date-range-picker__time-header" v-if="showTime">
          <span class="el-date-range-picker__editors-wrap">
            <input
              placeholder="开始日期"
              class="el-date-range-picker__editor"
              v-model="leftVisibleDate"
              @input="handleDateInput($event, 'min')"
              @change="handleDateChange($event, 'min')"/>
            <span class="el-date-range-picker__time-picker-wrap">
              <input
                placeholder="开始时间"
                class="el-date-range-picker__editor"
                v-model="leftVisibleTime"
                @focus="leftTimePickerVisible = true"
                @change="handleTimeChange($event, 'min')"/>
              <time-picker
                v-ref:lefttimepicker
                :date="minDate"
                @pick="handleLeftTimePick"
                v-show="leftTimePickerVisible">
              </time-picker>
            </span>
          </span>
          <span class="el-icon-arrow-right"></span>
          <span class="el-date-range-picker__editors-wrap is-right">
            <input
              placeholder="结束日期"
              class="el-date-range-picker__editor"
              v-model="rightVisibleDate"
              :readonly="!minDate"
              @input="handleDateInput($event, 'max')"
              @change="handleDateChange($event, 'max')" />
            <span class="el-date-range-picker__time-picker-wrap">
              <input
                placeholder="结束时间"
                class="el-date-range-picker__editor"
                v-model="rightVisibleTime"
                @focus="minDate && (rightTimePickerVisible = true)"
                :readonly="!minDate"
                @change="handleTimeChange($event, 'max')" />
              <time-picker
                v-ref:righttimepicker
                :date="maxDate"
                @pick="handleRightTimePick"
                v-show="rightTimePickerVisible"></time-picker>
            </span>
          </span>
        </div>
        <div class="el-picker-panel__content el-date-range-picker__content is-left">
          <div class="el-date-range-picker__header">
            <button
              @click="prevYear"
              class="el-picker-panel__icon-btn el-icon-d-arrow-left"></button>
            <button
              @click="prevMonth"
              class="el-picker-panel__icon-btn el-icon-arrow-left"></button>
            <div>{{ leftLabel }}</div>
          </div>
          <date-table
            selection-mode="range"
            :date="date"
            :year="leftYear"
            :month="leftMonth"
            :min-date="minDate"
            :max-date="maxDate"
            :range-state="rangeState"
            @changerange="handleChangeRange"
            @pick="handleRangePick">
          </date-table>
        </div>
        <div class="el-picker-panel__content el-date-range-picker__content is-right">
          <div class="el-date-range-picker__header">
            <button
              @click="nextYear"
              class="el-picker-panel__icon-btn el-icon-d-arrow-right"></button>
            <button
              @click="nextMonth"
              class="el-picker-panel__icon-btn el-icon-arrow-right"></button>
            <div>{{ rightLabel }}</div>
          </div>
          <date-table
            selection-mode="range"
            :date="rightDate"
            :year="rightYear"
            :month="rightMonth"
            :min-date="minDate"
            :max-date="maxDate"
            :range-state="rangeState"
            @changerange="handleChangeRange"
            @pick="handleRangePick"></date-table>
        </div>
      </div>
    </div>
    <div class="el-picker-panel__footer" v-if="showTime">
      <a
        href="JavaScript:"
        class="el-picker-panel__link-btn"
        @click="changeToToday">{{ $t('datepicker.today') }}</a>
      <button
        class="el-picker-panel__btn"
        @click="handleConfirm"
        :disabled="btnDisabled">确定</button>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import { nextMonth, prevMonth, $t, formatDate, parseDate } from '../util';

  export default {
    props: {
      date: {
        default() {
          return new Date();
        }
      },

      minDate: {},

      maxDate: {},

      rangeState: {
        default() {
          return {
            endDate: null,
            selecting: false,
            row: null,
            column: null
          };
        }
      },

      showTime: {
        type: Boolean,
        default: false
      },

      shortcuts: {},

      value: {},

      visible: Boolean
    },

    computed: {
      btnDisabled() {
        return !(this.minDate && this.maxDate && !this.selecting);
      },

      leftLabel() {
        return this.date.getFullYear() + '年 ' + (this.date.getMonth() + 1) + '月';
      },

      rightLabel() {
        return this.rightDate.getFullYear() + '年 ' + (this.rightDate.getMonth() + 1) + '月';
      },

      leftYear() {
        return this.date.getFullYear();
      },

      leftMonth() {
        return this.date.getMonth();
      },

      rightYear() {
        return this.rightDate.getFullYear();
      },

      rightMonth() {
        return this.rightDate.getMonth();
      },

      leftVisibleDate() {
        return formatDate(this.minDate);
      },

      rightVisibleDate() {
        return formatDate(this.maxDate);
      },

      leftVisibleTime() {
        return formatDate(this.minDate, 'HH:mm:ss');
      },

      rightVisibleTime() {
        return formatDate(this.maxDate, 'HH:mm:ss');
      },

      leftHours: {
        get() {
          return this.date.getHours();
        },
        set(hours) {
          this.date.setHours(hours);
        }
      },

      leftMinutes: {
        get() {
          return this.date.getMinutes();
        },
        set(minutes) {
          this.date.setMinutes(minutes);
        }
      },

      leftSeconds: {
        get() {
          return this.date.getSeconds();
        },
        set(seconds) {
          this.date.setSeconds(seconds);
        }
      },

      rightHours: {
        get() {
          return this.rightDate.getHours();
        },
        set(hours) {
          this.rightDate.setHours(hours);
        }
      },

      rightMinutes: {
        get() {
          return this.rightDate.getMinutes();
        },
        set(minutes) {
          this.rightDate.setMinutes(minutes);
        }
      },

      rightSeconds: {
        get() {
          return this.rightDate.getSeconds();
        },
        set(seconds) {
          this.rightDate.setSeconds(seconds);
        }
      },

      rightDate() {
        const newDate = new Date(this.date);
        const month = newDate.getMonth();
        newDate.setDate(1);

        if (month === 11) {
          newDate.setFullYear(newDate.getFullYear() + 1);
          newDate.setMonth(0);
        } else {
          newDate.setMonth(month + 1);
        }
        return newDate;
      }
    },

    data() {
      return {
        leftTimePickerVisible: false,
        rightTimePickerVisible: false
      };
    },

    watch: {
      minDate() {
        this.$nextTick(() => {
          if (this.maxDate && this.maxDate < this.minDate) {
            this.maxDate = null;
          }
        });
      },

      maxDate() {
      },

      leftTimePickerVisible(val) {
        if (val) this.$refs.lefttimepicker.ajustScrollTop();
      },

      rightTimePickerVisible(val) {
        if (val) this.$refs.righttimepicker.ajustScrollTop();
      },

      value(newVal) {
        if (!newVal) {
          this.minDate = null;
          this.maxDate = null;
        } else if (Array.isArray(newVal)) {
          this.minDate = newVal[0];
          this.maxDate = newVal[1];
        }
      }
    },

    methods: {
      $t,

      handleDateInput(event, type) {
        const value = event.target.value;
        const parsedValue = parseDate(value, 'yyyy-MM-dd');
        if (parsedValue) {
          const target = new Date(type === 'min' ? this.minDate : this.maxDate);
          if (target) {
            target.setFullYear(parsedValue.getFullYear());
            target.setMonth(parsedValue.getMonth());
            target.setDate(parsedValue.getDate());
          }
        }
      },

      handleChangeRange(val) {
        this.minDate = val.minDate;
        this.maxDate = val.maxDate;
        this.rangeState = val.rangeState;
      },

      handleDateChange(event, type) {
        const value = event.target.value;
        const parsedValue = parseDate(value, 'yyyy-MM-dd');
        if (parsedValue) {
          const target = new Date(type === 'min' ? this.minDate : this.maxDate);
          if (target) {
            target.setFullYear(parsedValue.getFullYear());
            target.setMonth(parsedValue.getMonth());
            target.setDate(parsedValue.getDate());
          }
          if (type === 'min') {
            if (target < this.maxDate) {
              this.minDate = new Date(target.getTime());
            }
          } else {
            if (target > this.minDate) {
              this.maxDate = new Date(target.getTime());
              if (this.minDate && this.minDate > this.maxDate) {
                this.minDate = null;
              }
            }
          }
        }
      },

      handleTimeChange(event, type) {
        const value = event.target.value;
        const parsedValue = parseDate(value, 'HH:mm:ss');
        if (parsedValue) {
          const target = new Date(type === 'min' ? this.minDate : this.maxDate);
          if (target) {
            target.setHours(parsedValue.getHours());
            target.setMinutes(parsedValue.getMinutes());
            target.setSeconds(parsedValue.getSeconds());
          }
          if (type === 'min') {
            if (target < this.maxDate) {
              this.minDate = new Date(target.getTime());
            }
          } else {
            if (target > this.minDate) {
              this.maxDate = new Date(target.getTime());
            }
          }
        }
      },

      handleRangePick(val) {
        this.maxDate = val.maxDate;
        this.minDate = val.minDate;
        if (!this.showTime) {
          this.$emit('pick', [this.minDate, this.maxDate]);
        }
      },

      changeToToday() {
        this.date = new Date();
      },

      handleShortcutClick(shortcut) {
        if (shortcut.onClick) {
          shortcut.onClick(this);
        }
      },

      resetView() {
        this.leftTimePickerVisible = false;
        this.rightTimePickerVisible = false;
      },

      handleLeftTimePick(value) {
        if (!this.minDate) {
          this.minDate = new Date();
        }
        this.minDate.setHours(value.getHours());
        this.minDate.setMinutes(value.getMinutes());
        this.minDate.setSeconds(value.getSeconds());

        this.minDate = new Date(this.minDate);
        this.leftTimePickerVisible = false;
      },

      handleRightTimePick(value) {
        if (!this.maxDate) {
          const now = new Date();
          if (now >= this.minDate) {
            this.maxDate = new Date();
          } else {

          }
        }

        if (this.maxDate) {
          this.maxDate.setHours(value.getHours());
          this.maxDate.setMinutes(value.getMinutes());
          this.maxDate.setSeconds(value.getSeconds());

          this.maxDate = new Date(this.maxDate);
        }

        this.rightTimePickerVisible = false;
      },

      prevMonth() {
        this.date = prevMonth(this.date);
      },

      nextMonth() {
        this.date = nextMonth(this.date);
      },

      nextYear() {
        const date = this.date;
        date.setFullYear(date.getFullYear() + 1);
        this.resetDate();
      },

      prevYear() {
        const date = this.date;
        date.setFullYear(date.getFullYear() - 1);
        this.resetDate();
      },

      handleConfirm() {
        this.$emit('pick', [this.minDate, this.maxDate]);
      },

      resetDate() {
        this.date = new Date(this.date);
      }
    },

    components: {
      TimePicker: require('./time'),
      DateTable: require('../basic/date-table')
    }
  };
</script>