<template>
  <div :class="{'input-group' : bootstrapStyling}" style="position:relative">
    <!-- Calendar Button -->
    <span v-if="calendarButton" class="vdp-datepicker__calendar-button" :class="{'input-group-prepend' : bootstrapStyling}" @click="showCalendar" v-bind:style="{'cursor:not-allowed;' : disabled}">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <input
      :type="inline ? 'hidden' : 'text'"
      :class="computedInputClass"
      :name="name"
      :ref="refName"
      :id="id"
      :value="formattedValue"
      :open-date="openDate"
      :placeholder="placeholder"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"
      maxlength="10"
      autocomplete="off">
    <!-- Clear Button -->
    <span v-if="clearButton && selectedDate" class="vdp-datepicker__clear-button" :class="{'input-group-append' : bootstrapStyling}" @click="clearDate()">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="clearButtonIcon">
          <span v-if="!clearButtonIcon">&times;</span>
        </i>
      </span>
    </span>
    <div style="position:absolute;left:0;top:0;height:100%;display:flex;align-items:center;padding:.75em .5em;pointer-events:none" v-if="typeable&&isSuggestion">
      <span style="opacity:0">{{suggestion}}</span>&nbsp;<span style="opacity:0.5">{{suggestion2}}</span>
    </div>
    <slot name="afterDateInput"></slot>
  </div>
</template>
<script>
import { makeDateUtils } from '../utils/DateUtils'
export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: String,
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      input: null,
      typedDate: false,
      utils: constructedDateUtils,
      isSuggestion: true,
      suggestion: '',
      suggestion2: 'MM/DD/YYYY'
    }
  },
  computed: {
    formattedValue () {
      // set suggestion default state
      this.isSuggestion = true
      if (!this.selectedDate) {
        return null
      }
      if (this.typedDate) {
        return this.typedDate
      }
      // don't show suggestion when pick a date
      this.isSuggestion = false
      return typeof this.format === 'function'
        ? this.format(this.selectedDate)
        : this.utils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    },

    computedInputClass () {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === 'string') {
          return [this.inputClass, 'form-control'].join(' ')
        }
        return {'form-control': true, ...this.inputClass}
      }
      return this.inputClass
    }
  },
  watch: {
    resetTypedDate () {
      this.typedDate = false
    }
  },
  methods: {
    showCalendar () {
      this.$emit('showCalendar')
    },
    getTypedDate (m, d, y) {
      var _m = parseInt(m) || 1
      var _d = parseInt(d) || 1

      var _y = parseInt(y)
      if (isNaN(_y)) {
        _y = new Date().getFullYear()
      } else {
        if (y.length === 4) {
          var year = new Date('01/01/' + y).getFullYear()
          if (_y < year) {
            y = _y = year
          }
        }
      }

      if (_m > 12) {
        _m = 12
      }
      var maxDay = this.utils.daysInMonth(_m, _y)
      if (_d > maxDay) {
        _d = maxDay
      }

      var arr = [_m, _d, _y]
      return {
        real: arr.join('/'),
        view: [m, d, y].filter(function (s) {
          return s !== ''
        }).map(function (s, index) {
          // format mm and dd
          if (index < 2) {
            if (s === '00') {
              return '01'
            } else if (parseInt(s) > arr[index]) {
              return arr[index]
            }
          }
          return s
        }).join('/')
      }
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate (event) {
      // close calendar if escape or enter are pressed
      if ([
        27, // escape
        13 // enter
      ].includes(event.keyCode)) {
        this.input.blur()
      }

      if (this.typeable) {
        // get input's number
        var arr = this.input.value.split('').filter(function (char) {
          return /\d/.test(char)
        })
        // get typedDate
        var m = arr.splice(0, 2).join('')
        var d = arr.splice(0, 2).join('')
        var y = arr.splice(0, 4).join('')
        var td = this.getTypedDate(m, d, y)

        var value = td.view
        this.suggestion = value
        this.suggestion2 = 'MM/DD/YYYY'.substring(value.length)
        this.input.value = value

        if (value === '') {
          this.clearDate()
          this.typedDate = null
          return
        }

        this.typedDate = value
        this.$emit('typedDate', new Date(td.real))
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred () {
      if (this.typeable && isNaN(Date.parse(this.input.value))) {
        this.clearDate()
        this.input.value = null
        this.typedDate = null
      }

      this.$emit('closeCalendar')
    },
    /**
     * emit a clearDate event
     */
    clearDate () {
      this.$emit('clearDate')
    }
  },
  mounted () {
    this.input = this.$el.querySelector('input')
  }
}
// eslint-disable-next-line
;
</script>
