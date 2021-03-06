<template>
  <div class="slider" :class="{ disabled }">
    <div class="runway" :style="runwayStyle" ref="slider" @click="handleSliderClick">
      <div class="bar" :style="barStyle"></div>
      <div class="dot" :style="dotStyle"
        @mousedown="handleMouseDown">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Slider',
    props: {
      value: {
        type: Number,
        default: 0
      },
      height: {
        type: Number,
        default: 4
      },
      color: {
        type: Boolean,
        default: true
      },
      disabled: {
        type: Boolean,
        default: false
      }
    },
    data () {
      return {
        dragging: false,
        dragStartX: 0,
        dragStartPercent: 0,
        percent: this.value,
        sliderSize: null
      }
    },
    computed: {
      runwayStyle () {
        return {
          height: this.height + 'px',
          borderRadius: this.height / 2 + 'px'
        }
      },
      barStyle () {
        return {
          width: this.percent + '%',
          backgroundColor: !this.color && 'rgba(255, 255, 255, .6)',
          borderRadius: this.height / 2 + 'px'
        }
      },
      dotStyle () {
        const width = this.height + 6
        return {
          width: width + 'px',
          height: width + 'px',
          left: this.percent + '%'
        }
      }
    },
    watch: {
      value (val) {
        if (!this.dragging) {
          this.percent = this.checkPercent(val)
        }
      }
    },
    mounted () {
      this.setSize()
      window.addEventListener('resize', this.resetSize, { passive: true })
    },
    beforeDestroy () {
      window.removeEventListener('resize', this.resetSize)
    },
    methods: {
      checkPercent (percent) {
        if (percent <= 0) {
          return 0
        } else if (percent >= 100) {
          return 100
        }
        return percent
      },
      setSize () {
        if (this.$refs.slider) {
          this.sliderSize = this.$refs.slider.clientWidth
        }
      },
      handleSliderClick (e) {
        if (this.dragging || this.disabled) {
          return
        }
        this.setSize()
        this.percent = e.layerX / this.$refs.slider.clientWidth * 100
        this.$emit('change', this.percent)
      },
      handleMouseDown (e) {
        if (this.disabled) {
          return
        }
        this.onDragStart(e)
        window.addEventListener('mousemove', this.onDragging, { passive: true })
        window.addEventListener('mouseup', this.onDragEnd, { passive: true })
        window.addEventListener('contextmenu', this.onDragEnd, { passive: true })
      },
      onDragStart (e) {
        this.dragging = true
        this.dragStartX = e.clientX
        this.dragStartPercent = this.percent
      },
      onDragging (e) {
        if (this.dragging) {
          const diff = (e.clientX - this.dragStartX) / this.$refs.slider.clientWidth * 100
          this.percent = this.checkPercent(this.dragStartPercent + diff)
        }
      },
      onDragEnd (e) {
        if (this.dragging) {
          setTimeout(() => {
            this.dragging = false
            this.dragStartPercent = 0
            this.dragStartX = 0
            this.$emit('change', this.percent)
          }, 0)
          window.removeEventListener('mousemove', this.onDragging)
          window.removeEventListener('mouseup', this.onDragEnd)
          window.removeEventListener('contextmenu', this.onDragEnd)
        }
      }
    }
  }
</script>

<style lang="stylus" scoped>
  @import '~assets/stylus/_var'
  @import '~assets/stylus/_mixin'

  .slider {
    width 100%

    .runway {
      position relative
      width 100%
      background alpha($white, .2)
      cursor pointer

      .bar {
        left 0
        height 100%
        background $base-color
      }

      .dot {
        position absolute
        top -3px
        border-radius 100%
        background $white
        transform translateX(-50%)
        transition transform .3s $ease, box-shadow .3s $ease

        &:hover {
          box-shadow 0 0 10px 0 alpha($white, 1)
          transform translateX(-50%) scale(1.1)
        }
      }
    }

    &.disabled {
      opacity .8
      .runway
      .dot {
        cursor not-allowed !important
      } 
    }
  }
</style>
