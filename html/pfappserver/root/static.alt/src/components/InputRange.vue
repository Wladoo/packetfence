<template>
  <div class="w-100 input-range"> <!-- external style applied here -->
    <div
      :disabled="disabled"
      :min="parseFloat(inputValue) === parseFloat(min)"
      :max="parseFloat(inputValue) === parseFloat(max)"
      :index="inputValue"
      :style="[{ 'width': `${width}px` }, ((color) ? { '--range-background-color': color } : {})]"
      class="range"
    >
      <div>
        <div v-for="(hint, index) in hints" :key="index" class="hint" :style="hintStyle(index)"></div>
        <span class="handle" :style="{ left: `${percent(inputValue)}%` }">
          <slot/> <!-- Icon slot -->
        </span>
        <div v-if="label" class="label" :style="(inputValue >= ((max - min) / 2)) ? { 'justify-content': 'flex-start' } : { 'justify-content': 'flex-end' }">
          {{ label }}
        </div>
        <div v-if="tooltip" class="tooltip" :style="{ left: `${percent(inputValue)}%` }">
          <span id="value">{{ $t(tooltipFunction(inputValue)) }}</span>
        </div>
      </div>
      <input
        v-on="forwardListeners"
        type="range"
        :tabindex="tabIndex"
        :value="inputValue"
        :max="max"
        :min="min"
        :step="step"
        :disabled="disabled"
        @input="clickInput"
      />
      <div v-if="!disabled" class="catch-min" @click.stop.prevent="clickMin($event)"><!-- catch click left of input --></div>
      <div v-if="!disabled" class="catch-max" @click.stop.prevent="clickMax($event)"><!-- catch click right of input --></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'input-range',
  props: {
    value: {
      default: null
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    },
    disabled: {
      type: Boolean,
      default: false
    },
    color: { /* override default colors via JS */
      type: String,
      default: null
    },
    label: { /* inner label, flips left/right @ +/- 50% */
      type: String,
      default: null
    },
    tooltip: { /* set to `true` to enable tooltips */
      type: Boolean,
      default: false
    },
    tooltipFunction: { /* tooltip string callback function */
      type: Function,
      default: (value) => { return value }
    },
    hints: { /* dots/pills in range for hints (eg: [1, [1-2], 2]) */
      type: Array,
      default: () => { return [] }
    },
    listenInput: { /* disable to track events manually (eg: toggle) */
      type: Boolean,
      default: true
    },
    width: {
      type: Number,
      default: 40
    },
    tabIndex: {
      type: Number,
      default: 0
    }
  },
  computed: {
    inputValue: {
      get () {
        return this.value
      },
      set (newValue) {
        this.$emit('input', newValue)
      }
    },
    forwardListeners () {
      const { input, ...listeners } = this.$listeners
      return listeners
    }
  },
  methods: {
    clickInput ($event) {
      if (this.disabled) return
      if (this.listenInput) {
        this.$set(this, 'inputValue', $event.target.value)
      }
    },
    clickMin ($event) {
      this.$set(this, 'inputValue', this.min)
    },
    clickMax ($event) {
      this.$set(this, 'inputValue', this.max)
    },
    percent (value = this.inputValue) {
      if (value >= this.max) return 100
      if (value <= this.min) return 0
      return (100 / (this.max - this.min)) * parseInt(value) - (100 / (this.max - this.min)) * this.min
    },
    hintStyle (index) {
      let style = {}
      if (index in this.hints) {
        const hint = this.hints[index]
        if (hint.constructor === Array) { // range
          style.left = `${this.percent(hint[0])}%`
          style.width = `calc(${this.percent(hint[1] - hint[0])}% + var(--handle-height))`
        } else { // single
          style.left = `${this.percent(hint)}%`
          style.width = 'var(--handle-height)'
        }
        return style
      }
    }
  }
}
</script>

<style lang="scss">
:root { /* defaults */
  --range-height: 22px;
  --range-background-color: #{$input-placeholder-color};
  --range-transition-delay: 0.3s;
  --handle-height: 16px;
  --handle-background-color: var(--white);
  --hint-background-color: var(--light);
  --tooltip-transition-delay: 0.3s;
}

@keyframes animateHint {
  from { opacity: 0; left: 50%; width: var(--handle-height); }
  to { opacity: 0.6; }
}

.range {
  position: relative;
  height: var(--range-height);
  margin: 0px;
  box-shadow: 0 0 0 1px transparent; /* pseudo border */
  border-radius: calc(var(--range-height) / 2);
  background-color: var(--range-background-color, $input-placeholder-color);
  text-align: left;
  transition: background-color var(--range-transition-delay) ease-out,
    box-shadow var(--range-transition-delay) ease-out,
    outline var(--range-transition-delay) ease-out;
  > div {
    position: absolute;
    right: calc(var(--range-height) / 2);
    left: calc(var(--range-height) / 2);
    height: var(--range-height);
    > .handle {
      position: absolute;
      top: calc((var(--range-height) - var(--handle-height)) / 2);
      display: flex;
      justify-content: center;
      align-items: center;
      width: var(--handle-height);
      height: var(--handle-height);
      margin-left: calc(var(--handle-height) / -2);
      border-radius: 50%;
      outline: none;
      background-color: var(--handle-background-color);
      color: var(--range-background-color, $input-placeholder-color); /* SVG icon */
      font-size: 10px;
      text-align: left;
      transition: left var(--handle-transition-delay, 0s) ease-in-out, /* do not animate `left` unless explicit */
        background-color var(--range-transition-delay) ease-out,
        color var(--range-transition-delay) ease-out;
    }
    > .hint {
      position: absolute;
      top: calc((var(--range-height) - var(--handle-height)) / 2);
      height: var(--handle-height);
      margin-left: calc(var(--handle-height) / -2);
      background-color: var(--hint-background-color);
      border-top-left-radius: var(--handle-height) 100%;
      border-bottom-left-radius: var(--handle-height) 100%;
      border-top-right-radius: var(--handle-height) 100%;
      border-bottom-right-radius: var(--handle-height) 100%;
      animation: animateHint var(--handle-transition-delay);
      transition: background-color var(--range-transition-delay) ease-out;
    }
    > .label {
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      display: flex;
      align-items: center;
      color: var(--hint-background-color);
    }
    > .tooltip {
      position: absolute;
      bottom: calc(50% + (var(--handle-height) / 2) + 12px);
      justify-content: center;
      align-items: center;
      width: auto;
      opacity: 0;
      visibility: hidden;
      transform: translateX(-50%);
      min-width: var(--handle-height);
      font-size: .7875rem;
      text-align: center;
      transition: left var(--handle-transition-delay, 0s) ease-in-out, /* do not animate `left` unless explicit */
        width var(--handle-transition-delay, 0s) ease-in-out, /* do not animate `width` unless explicit */
        visibility var(--tooltip-transition-delay) linear,
        opacity var(--tooltip-transition-delay) ease-in-out;
      &:after { /* tooltip arrow */
        content: "";
        position: absolute;
        top: 20px;
        left: 50%;
        border-color: transparent;
        border-top-color: #000;
        border-style: solid;
        border-width: .4rem .4rem 0;
        transform: translateX(-50%);
      }
      > span { /* tooltip body */
        padding: .25rem .5rem;
        background-color: #000;
        color: #fff;
        border-radius: .25rem;
        font-family: var(--font-family-sans-serif);
        font-size: .7875rem;
        font-style: normal;
        font-weight: 400;
        line-height: 1.5;
        letter-spacing: normal;
        text-align: center;
        text-shadow: none;
        text-transform: none;
        white-space: nowrap;
        word-break: normal;
        word-spacing: normal;
        line-break: auto;
        &::selection { background: transparent; }
        &::-moz-selection { background: transparent; }
      }
    }
  }
  > input[type=range] {
    position: absolute;
    top: calc((var(--range-height) - var(--handle-height)) / 2);
    right: calc(var(--range-height) / 2);
    left: calc(var(--range-height) / 2);
    width: calc(100% - var(--range-height));
    pointer-events: all;
    -webkit-appearance: none; /* disable track clicks */
    height: var(--handle-height);
    opacity: 0;
    &::-ms-track {
      -webkit-appearance: none;
      background: transparent;
      color: transparent;
    }
    &::-moz-range-track {
      -moz-appearance: none;
      background: transparent;
      color: transparent;
    }
    &:focus::-webkit-slider-runnable-track {
      border: transparent;
      background: transparent;
    }
    &:focus {
      outline: none;
    }
    &::-ms-thumb {
      pointer-events: all;
      width: var(--range-height);
      height: var(--range-height);
      border-radius: 0px;
      border: 0 none;
      background: red;
    }
    &::-moz-range-thumb {
      pointer-events: all;
      width: var(--range-height);
      height: var(--range-height);
      border-radius: 0px;
      border: 0 none;
      background: red;
    }
    &::-webkit-slider-thumb {
      pointer-events: all;
      width: var(--range-height);
      height: var(--range-height);
      border-radius: 0px;
      border: 0 none;
      background: red;
      -webkit-appearance: none;
    }
    &::-ms-fill-lower {
      border: 0 none;
      background: transparent;
    }
    &::-ms-fill-upper {
      border: 0 none;
      background: transparent;
    }
    &::-ms-tooltip {
      display: none;
    }
  }
  > .catch-min {
    position: absolute;
    top: calc((var(--range-height) - var(--handle-height)) / 2);
    left: 0px;
    width: calc(var(--range-height) / 2);
    height: var(--handle-height);
  }
  > .catch-max {
    position: absolute;
    top: calc((var(--range-height) - var(--handle-height)) / 2);
    left: calc(100% - (var(--range-height) / 2));
    width: calc(var(--range-height) / 2);
    height: var(--handle-height);
  }
  > div > .hint,
  &[disabled] {
    opacity: 0.6;
  }
  &:not([disabled]) {
    &:not([min]) > .catch-min {
      cursor: move; /* fallback if w-resize cursor is unsupported */
      cursor: w-resize;
    }
    &:not([max]) > .catch-max {
      cursor: move; /* fallback if e-resize cursor is unsupported */
      cursor: e-resize;
    }
    &:hover > div > .tooltip {
      opacity: 1;
      visibility: visible;
    }
    > input[type="range"] {
      cursor: move; /* fallback if grab cursor is unsupported */
      cursor: grab;
    }
  }
}
</style>
