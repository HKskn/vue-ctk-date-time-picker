<template>
  <div
    :class="{'is-dark': dark}"
    class="shortcuts-container"
  >
    <button
      v-for="shortcut in shortcuts"
      :style="[shortcut.isSelected ? getSelectedStyle : getStyle]"
      :key="shortcut.value"
      :class="{ 'is-selected': shortcut.isSelected }"
      class="shortcut-button"
      tabindex="-1"
      @mouseover="shortcut.isHover = true"
      @mouseleave="shortcut.isHover = false"
      @click="select(shortcut)"
    >
      <span
        :style="[getEffectStyle]"
        class="datepicker-button-effect"
      />
      <span class="shortcut-button-content">
        {{ shortcut.label }}
      </span>
    </button>
  </div>
</template>
<script>
  import moment from 'moment'

  export default {
    name: 'CtkCalendarShortcut',
    props: {
      value: { type: Object, required: false, default: null },
      color: { type: String, default: String },
      locale: { type: String, default: String },
      dark: { type: Boolean, default: false },
      dateTime: {type: Object, default: Object},
      customShortcuts: {type: Array, default: () => []}
    },
    data () {
      return {
        types: ['day', '-day', 'isoWeek', '-isoWeek', 'month', '-month', 'year', '-year'],
        shortcuts: [
          { label: 'Today', value: 'day', isHover: false, isSelected: false },
          { label: 'Yesterday', value: '-day', isHover: false, isSelected: false },
          { label: 'This week', value: 'isoWeek', isHover: false, isSelected: false },
          { label: 'Last week', value: '-isoWeek', isHover: false, isSelected: false },
          { label: 'Last 7 days', value: 7, isHover: false, isSelected: false },
          { label: 'Last 30 days', value: 30, isHover: false, isSelected: false },
          { label: 'This month', value: 'month', isHover: false, isSelected: false },
          { label: 'Last month', value: '-month', isHover: false, isSelected: false },
          { label: 'This year', value: 'year', isHover: false, isSelected: false },
          { label: 'Last year', value: '-year', isHover: false, isSelected: false }
        ],
        computedTypes: {}
      }
    },
    computed: {
      getStyle () {
        return {
          color: this.color
        }
      },
      getSelectedStyle () {
        return {
          backgroundColor: this.color,
          color: '#FFF'
        }
      },
      getEffectStyle () {
        return {
          backgroundColor: this.color
        }
      }
    },
    watch: {
      customShortcuts (newVal, oldVal) {
        if (newVal && Array.isArray(newVal) && JSON.stringify(newVal) !== JSON.stringify(oldVal)) {
          this.init()
        }
      }
    },
    created () {
      this.init()
    },
    methods: {
      init () {
        this.overrideCustomShortCuts()
        this.preparePreCalculatedTypes()

        let selected = this.shortcuts.filter(shortcut => shortcut.isSelected)
        if (selected[0]) {
          this.select(selected[0])
        }
      },
      preparePreCalculatedTypes () {
        this.shortcuts.forEach(shortcut => {
          const { value } = shortcut

          switch (true) {
          case value === 'isoWeek': case value === 'month': case value === 'year': case value === 'day':
            shortcut.start = moment().locale(this.locale).startOf(value)
            shortcut.end = moment().locale(this.locale).endOf(value)
            break
          case typeof value === 'number':
            shortcut.end = moment().locale(this.locale)
            shortcut.start = moment().locale(this.locale).subtract(value, 'd')
            break
          case value === '-month':
            shortcut.start = moment().locale(this.locale).subtract(1, 'months').startOf('month')
            shortcut.end = moment().locale(this.locale).subtract(1, 'months').endOf('month')
            break
          case value === '-year':
            shortcut.start = moment().locale(this.locale).subtract(1, 'years').startOf('year')
            shortcut.end = moment().locale(this.locale).subtract(1, 'years').endOf('year')
            break
          case value === '-isoWeek':
            shortcut.start = moment().locale(this.locale).subtract(1, 'weeks').startOf('week')
            shortcut.end = moment().locale(this.locale).subtract(1, 'weeks').endOf('week')
            break
          case value === '-day':
            shortcut.start = moment().locale(this.locale).subtract(1, 'days').startOf('day')
            shortcut.end = moment().locale(this.locale).subtract(1, 'days').endOf('day')
            break
          }
        })
      },
      overrideCustomShortCuts () {
        let customShortCuts = []
        this.customShortcuts.forEach(customShortcut => {
          if (this.isValidValue(customShortcut.value) && customShortcut.label) {
            customShortCuts.push({
              label: customShortcut.label,
              value: customShortcut.value,
              isHover: !!customShortcut.isHover,
              isSelected: !!customShortcut.isSelected
            })
          }
        })
        if (customShortCuts.length) {
          this.shortcuts = customShortCuts
        }
      },
      unSelectAllShortcuts () {
        this.shortcuts.forEach(sc => {
          sc.isSelected = false
        })
      },
      selectShortcut (shortcut) {
        this.unSelectAllShortcuts()
        shortcut.isSelected = true
      },
      select (shortcut) {
        this.selectShortcut(shortcut)
        const { start, end, value } = shortcut
        this.$emit('change-range', { start, end, value })
      },
      isValidValue (value) {
        return value && (this.types.indexOf(value) > -1 || (typeof value === 'number' && value > 0))
      }
    }
  }
</script>

<style lang="scss" scoped>
  .shortcuts-container {
    width: 160px;
    padding: 10px 5px;
    button.shortcut-button {
      width: 100%;
      position: relative;
      margin-bottom: 10px;
      border: 1px solid #eaeaea;
      height: 30px;
      font-weight: 300;
      line-height: 26px;
      border-radius: 30px;
      font-size: 12px;
      outline: none;
      cursor: pointer;
      -webkit-transition: all 0.25s cubic-bezier(0.645, 0.045, 0.355, 1);
      .datepicker-button-effect {
        position: absolute;
        background: #00C853;
        top: 0;
        right: 0;
        height: 30px;
        border-radius: 30px;
        width: 100%;
        -webkit-transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
        transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
        transform: scale(0);
        opacity: .6;
      }
      .shortcut-button-content {
        position: relative;
      }
      &:hover {
        color: #FFF !important;
        border-color: transparent;
        .datepicker-button-effect {
          transform: scale(1);
        }
      }
      &.is-selected {
        border-color: transparent;
        .datepicker-button-effect {
          opacity: 1;
          transform: scale(1);
        }
      }
    }
    &.is-dark {
      button.shortcut-button {
        background: #424242;
        border-color: lighten(#424242, 20%);
        .shortcut-button-content {
          color: #FFF;
        }
        &:hover, &.is-selected {
          border-color: transparent;
        }
      }
    }
  }

  @media screen and (max-width: 412px) {
    .shortcuts-container:not(.inline) {
      width: 100%;
      border-bottom: 1px solid #EAEAEA;
      height: 120px !important;
      overflow: auto;
    }
  }
</style>
