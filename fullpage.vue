<template>
<section :id="'fullpage_section_' + name" class="fullpage_section" ref="fullpage_section">
  <slot/>
</section>
</template>

<script>
import smoothscroll from 'smoothscroll-polyfill'

export default {
  props: {
    name: {
      type: String,
      required: true
    },
    accuracy: {
      type: Number,
      default: 20
    },
    startAt: {
      default: 0
    },
    duration: {
      type: Number,
      default: 300
    }
  },
  data() {
    return {
      section: 0,
      finishAt: 0,
      frame: 0,
      touch: []
    }
  },
  computed: {
    box() {
      return this.$refs.fullpage_section
    },
    sections() {
      return this.$refs.fullpage_section.children
    }
  },
  methods: {
    addEvent (n) {
      this.sections[n].addEventListener('wheel', e => this.checkWheelDirection(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].addEventListener('touchstart', e => this.writeTouchPoint(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].addEventListener('touchend', e => this.checkTouchDirection(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].addEventListener('mousedown', e => this.writeDragPoint(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].addEventListener('mouseup', e => this.checkDragDirection(e, n), {
        capture: true,
        passive: true
      })
    },
    removeEvent (n) {
      this.sections[n].removeEventListener('wheel', e => this.checkWheelDirection(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].removeEventListener('touchstart', e => this.writeDragPoint(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].removeEventListener('touchend', e => this.checkTouchDirection(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].removeEventListener('mousedown', e => this.writeDragPoint(e, n), {
        capture: true,
        passive: true
      })
      this.sections[n].removeEventListener('mouseup', e => this.checkDragDirection(e, n), {
        capture: true,
        passive: true
      })
    },
    addEventAll () {
      for (let i = 0; i < this.sections.length; i++) this.addEvent(i)
    },
    removeEventAll () {
      for (let i = 0; i < this.sections.length; i++) this.removeEvent(i)
    },
    checkWheelDirection (e, n) {
      const date = new Date().getTime()
      if(this.finishAt < date) {
        let v = n
        if (e.deltaY < 0 || e.deltaX < 0) v = n - 1
        else if (e.deltaY > 0 || e.deltaX > 0) v = n + 1
        if (v < 0) v = 0
        if (v > this.sections.length - 1) v = this.sections.length - 1
        this.section = v
      }
    },
    writeTouchPoint (e, n) {
      this.touch[n] = {
        x: e.touches[0].clientX,
        y: e.touches[0].clientY
      }
    },
    checkTouchDirection (e, n) {
      const deltaX = e.changedTouches[0].clientX - this.touch[n].x
      const deltaY = e.changedTouches[0].clientY - this.touch[n].y
      let v = n
      if (Math.abs(deltaY) > this.accuracy || Math.abs(deltaX) > this.accuracy) {
        if (Math.abs(deltaY) >= Math.abs(deltaX)) {
          if (deltaY < 0) v = n + 1
          else if (deltaY > 0) v = n - 1
        } else {
          if (deltaX < 0) v = n + 1
          else if (deltaX > 0) v = n - 1
        }
      }
      if (v < 0) v = 0
      if (v > this.sections.length - 1) v = this.sections.length - 1
      this.section = v
    },
    writeDragPoint (e, n) {
      this.touch[n] = {
        x: e.clientX,
        y: e.clientY
      }
    },
    checkDragDirection (e, n) {
      const deltaX = e.clientX - this.touch[n].x
      const deltaY = e.clientY - this.touch[n].y
      let v = n
      if (Math.abs(deltaY) > this.accuracy || Math.abs(deltaX) > this.accuracy) {
        if (Math.abs(deltaY) >= Math.abs(deltaX)) {
          if (deltaY < 0) v = n + 1
          else if (deltaY > 0) v = n - 1
        } else {
          if (deltaX < 0) v = n + 1
          else if (deltaX > 0) v = n - 1
        }
      }
      if (v < 0) v = 0
      if (v > this.sections.length - 1) v = this.sections.length - 1
      this.section = v
    },
    setAnimationFrame (f) {
      window.requestAnimationFrame = window.requestAnimationFrame ||
      window.mozRequestAnimationFrame ||
      window.webkitRequestAnimationFrame ||
      window.msRequestAnimationFrame ||
      function (f) {
        setTimeout(f, 1000 / 60)
      }
    },
    scrollAnimation (n, p) {
      this.removeEventAll()
      const date = new Date().getTime()
      if (this.finishAt <= date) {
        this.box.scroll({
          top: this.sections[n].offsetTop,
          left: 0
        })
        this.frame = 0
        this.$emit('onchange', {
          ready: true,
          prev: p,
          next: n
        })
        this.addEvent(this.section)
      } else {
        try {
          const timeleft = this.finishAt - date
          this.frame = timeleft
          const ny = this.sections[n].offsetTop
          const py = this.sections[p].offsetTop
          const nx = this.sections[n].offsetLeft
          const px = this.sections[p].offsetLeft
          const frameY = (ny - py) / this.duration
          const frameX = (nx - px) / this.duration
          const queue = this.duration - timeleft
          const npY = py + (queue * frameY)
          const npX = px + (queue * frameY)
          this.box.scroll({
            top: npY,
            left: npX
          })
          window.requestAnimationFrame(() => this.scrollAnimation(n, p))
        } catch (e) {}
      }
    }
  },
  watch: {
    section: function (n, p) {
      this.finishAt = new Date().getTime() + this.duration
      this.scrollAnimation(n, p)
      this.$emit('onchange', {
        ready: false,
        prev: p,
        next: n
      })
    }
  },
  mounted() {
    smoothscroll.polyfill()
    this.section = this.startAt
    this.box.addEventListener('resize', () => {
      this.box.scroll({
        top: this.sections[this.section].offsetTop,
        left: 0
      })
    })
    this.setAnimationFrame()
    this.addEventAll()
  },
  beforeDestroy() {
    this.removeEventAll()
    this.box.removeEventListener('resize', () => {
      this.box.scroll({
        top: this.sections[this.section].offsetTop,
        left: 0
      })
    })
  }
}
</script>


<style>
.fullpage_section {
  position: fixed;
  overflow: hidden;
  height: 100%;
  width: 100%;
}
.fullpage_section>section, .fullpage_section>div {
  width: 100%;
  height: 100%;
}
</style>