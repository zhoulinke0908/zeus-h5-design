<template>
  <div class="slider-wrap">
    <div class="slider-track" :class="{ active: dragging, success: done }" ref="track">
      <div class="slider-mask" :style="{ width: maskWidth }">
        <van-icon name="success" v-show="done" class="mask-check" />
      </div>
      <div class="slider-tips">{{ done ? '验证通过' : '请按住滑块拖动到最右边' }}</div>
      <div
        class="slider-knob"
        :class="{ success: done }"
        ref="knob"
        @mousedown="onStart"
        @touchstart.prevent="onStart"
      >
        <van-icon name="arrow" v-show="!done" class="knob-arrow" />
        <van-icon name="success" v-show="done" class="knob-check" />
      </div>
    </div>
  </div>
</template>

<script>
const PRIMARY = '#F19871'
const SUCCESS = '#34C759'

export default {
  name: 'SliderVerify',
  data() {
    return {
      dragging: false,
      done: false,
      startX: 0,
      startLeft: 0,
      maskWidth: '0'
    }
  },
  methods: {
    maxLeft() {
      const track = this.$refs.track
      const knob = this.$refs.knob
      return track.offsetWidth - knob.offsetWidth - 4
    },
    onStart(e) {
      if (this.done) return
      this.dragging = true
      this.startX = e.touches ? e.touches[0].clientX : e.clientX
      this.startLeft = parseFloat(this.$refs.knob.style.left || 0)
    },
    onMove(e) {
      if (!this.dragging) return
      const cx = e.touches ? e.touches[0].clientX : e.clientX
      const dx = cx - this.startX
      const nl = Math.max(0, Math.min(this.startLeft + dx, this.maxLeft()))
      this.$refs.knob.style.left = nl + 'px'
      this.maskWidth = (nl + this.$refs.knob.offsetWidth / 2) + 'px'
    },
    onEnd() {
      if (!this.dragging) return
      this.dragging = false
      const cl = parseFloat(this.$refs.knob.style.left || 0)
      if (cl >= this.maxLeft() - 6) {
        this.$refs.knob.style.left = this.maxLeft() + 'px'
        this.maskWidth = '100%'
        this.done = true
      } else {
        this.$refs.knob.style.left = '0'
        this.maskWidth = '0'
      }
    },
    isDone() {
      return this.done
    },
    reset() {
      this.done = false
      this.dragging = false
      this.$refs.knob.style.left = '0'
      this.maskWidth = '0'
    }
  },
  mounted() {
    document.addEventListener('mousemove', this.onMove)
    document.addEventListener('touchmove', this.onMove, { passive: false })
    document.addEventListener('mouseup', this.onEnd)
    document.addEventListener('touchend', this.onEnd)
  },
  beforeDestroy() {
    document.removeEventListener('mousemove', this.onMove)
    document.removeEventListener('touchmove', this.onMove)
    document.removeEventListener('mouseup', this.onEnd)
    document.removeEventListener('touchend', this.onEnd)
  }
}
</script>

<style scoped>
.slider-wrap { margin-top: 24px; }
.slider-track {
  position: relative; width: 100%; height: 50px;
  background: #F5F5F7; border-radius: 25px;
  overflow: hidden; user-select: none;
  border: 1.5px solid #F2F2F5;
}
.slider-track.active { border-color: #F19871; }
.slider-track.success {
  border-color: #34C759; background: #F0FFF4;
}
.slider-mask {
  position: absolute; left: 0; top: 0; height: 100%;
  background: linear-gradient(90deg, rgba(241,152,113,0.15), rgba(241,152,113,0.25));
  border-radius: 25px; display: flex; align-items: center;
  justify-content: flex-end; padding-right: 28px;
}
.slider-track.success .slider-mask {
  background: linear-gradient(90deg, rgba(52,199,89,0.12), rgba(52,199,89,0.22));
}
.mask-check { font-size: 16px; color: #34C759; }
.slider-tips {
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  font-size: 14px; color: #86868B; pointer-events: none;
}
.slider-track.active .slider-tips,
.slider-track.success .slider-tips { opacity: 0; }

.slider-knob {
  position: absolute; left: 0; top: 50%; transform: translateY(-50%);
  width: 46px; height: 46px; background: #F19871;
  border-radius: 50%; display: flex; align-items: center;
  justify-content: center; cursor: grab; z-index: 2;
  box-shadow: 0 3px 10px rgba(241,152,113,0.35);
}
.slider-knob.success {
  background: #34C759;
  box-shadow: 0 3px 12px rgba(82,196,26,0.35);
}
.knob-arrow { font-size: 18px; color: #fff; }
.knob-check { font-size: 18px; color: #fff; }
</style>
