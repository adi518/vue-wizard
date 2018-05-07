<template>
  <div class="v-stepper">
    <v-step
      v-for="(step, $index) in map"
      :name="id"
      :key="$index"
      :debug="debug"
      :index="$index"
      :active="step.index === toIndex(value)"
      :visited="step.visited"
      :disabled="step.disabled"
      with-divider
      @change="onChange"
    >
      <template slot="index" slot-scope="api">
        <slot :name="`step-${api.displayIndex}-index`" v-bind="api">
          {{api.displayIndex}}
        </slot>
      </template>
      <template slot-scope="api">
        <slot :name="`step-${api.displayIndex}`" v-bind="api"></slot>
      </template>
    </v-step>
  </div>
</template>

<script>
// https://www.webpackbin.com/bins/-KvHS7KEmrTLJOYWS9k2
// https://cristijora.github.io/vue-form-wizard/#/?id=demos
// https://stackoverflow.com/questions/4852017/how-to-initialize-an-arrays-length-in-javascript

// Components
import VStep from './Step'

// Implementation
export default {
  name: 'VStepper',
  components: {
    VStep
  },
  inheritAttrs: false,
  props: {
    value: {
      type: Number,
      default: 1
    },
    steps: {
      type: Number,
      default: 0
    },
    linear: {
      type: Boolean,
      default: true
    },
    persist: {
      type: Boolean,
      default: false
    },
    storekeeper: {
      type: String,
      default: 'localStorage',
      validator(value) {
        return ['localStorage', 'sessionStorage'].includes(value)
      }
    },
    debug: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      map: this.getMap(),
      namespace: 'v-stepper',
      index: this.toIndex(this.value)
    }
  },
  watch: {
    value(value) {
      this.index = this.toIndex(value)
      if (this.persist) {
        this.setStorage()
      }
    },
    index: {
      handler(index) {
        this.$emit('input', this.toValue(index))
      },
      immediate: true
    }
  },
  created() {
    if (this.persist) {
      const storage = this.getStorage()
      if (storage) {
        this.map = storage.map
        this.index = storage.index
      } else {
        this.setStorage()
      }
    }
  },
  destroyed() {
    window[this.storekeeper].removeItem(this.id)
  },
  computed: {
    id() {
      return `${this.namespace}-${this._uid}`
    },
    lastIndex() {
      return this.map.length - 1
    }
  },
  methods: {
    toValue(index) {
      return index + 1
    },
    toIndex(value) {
      return value - 1
    },
    doesStepExist(index) {
      return !!this.map[index]
    },
    onChange(index, options = {}) {
      const isNext = index === this.index + 1
      const isPrevious = index === this.index - 1
      const oldIndex = this.toIndex(this.value)
      if (this.linear) {
        if (isNext || isPrevious) {
          this.setStep(index, 'active', true)
          this.setStep(index, 'visited', false)
          this.setStep(index, 'disabled', false)
          this.setStep(oldIndex, 'active', false)
          this.map.forEach(step => {
            if (step.index > index) {
              this.setStep(step.index, 'disabled', true)
            }
          })
          this.setStep(oldIndex, 'visited', true)
          this.$emit('input', this.toValue(index))
        }
      } else {
        this.setStep(oldIndex, 'visited', true)
        this.$emit('input', this.toValue(index))
      }
    },
    isIntermediateIndex(index) {
      return index < this.lastIndex
    },
    getMap() {
      return Array.from(Array(this.steps), (step, index) => {
        const isFirst = index === 0
        const isNext = index - 1 === 0
        let disabled = false
        if (this.linear) {
          if (isFirst || isNext) {
            // leave as is
          } else {
            disabled = true
          }
        }
        const visited = false
        const value = this.toValue(index)
        return { index, value, visited, disabled }
      })
    },
    offset(offset) {
      const index = this.index + offset
      if (this.doesStepExist(index)) {
        this.onChange(index, { emitAsIs: true })
      }
    },
    next() {
      this.offset(1)
    },
    previous() {
      this.offset(-1)
    },
    reset() {
      this.map = this.getMap()
      this.index = 0
      this.$emit('reset')
    },
    setStep(index, prop, value) {
      this.$set(this.map[index], prop, value)
    },
    setStorage() {
      const { index, map } = this
      window[this.storekeeper].setItem(this.id, JSON.stringify({ index, map }))
    },
    getStorage() {
      return JSON.parse(window[this.storekeeper].getItem(this.id))
    }
  }
}
</script>

<style lang="scss" scoped>
.v-stepper {
  display: flex;
  user-select: none;
  flex-direction: row;
  box-sizing: border-box;
  justify-content: space-between;

  @media only screen and (max-width: 640px) {
    flex-direction: column;
  }

  *,
  *::before,
  *::after {
    box-sizing: inherit;
  }
}
</style>