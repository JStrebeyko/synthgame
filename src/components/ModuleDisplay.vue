<template>

    <div class="display" ref="displayWrapper">
      <svg width="100%" height="100%">
        <rect :fill="fill" width="100%" height="100%" />
        <!-- <rect fill="black" width="100%" height="100%" /> -->

        <path stroke="black"
              :stroke-width="strokeWidth"
              :d="path"
              fill="black"
              style="fill-rule: nozero"
              />

        <path v-if="this.module === 'oscillator' || this.module === 'filter' || this.module === 'envelope'"
              stroke="white"
              v-show="!createModeIsActive"
              :stroke-width="2"
              :d="pathGoal"
              fill="none"
              style="fill-rule: nozero"
              />

        <!-- lfo: -->
        <!--positioning wrapperfix -->
        <svg :viewBox="lfoVB">

        <!-- the swing: -->
        <g v-if="this.module === 'lfo'" ref="swing"
          stroke="black"
          fill="black"
          :style="swingMovement"
          class="swingClass"
                          >
          <path
            :stroke-width="1"
            :fill="fill"
            :d="'m 0,'+ (-displayHeight/2)+
                ' v ' + (displayHeight)
                "/>

                <circle :cx="0" :cy="displayHeight*0.6" :r="displayHeight/4" />
          </g>

          <g v-if="this.module === 'lfo'"
            stroke="white"
            v-show="!createModeIsActive"
            fill="none"
            ref="swingGoal"
            :stroke-width="2"
            :style="goalSwingMovement"
            class="swingClass"
                            >
            <path
              :stroke-width="2"
              fill="none"
              stroke="white"
              :d="'m 0,'+ (-displayHeight/1.52)+
                  ' v ' + (displayHeight)
                  "/>

                  <circle :cx="0" :cy="displayHeight*0.6" :r="displayHeight/4"/>
            </g>
        </svg>
        <g v-if="debug" class="debug-text">
          <text x="0" y="0">
            <tspan x="0" y="10%">{{knobs[0].name}}: {{knobs[0].value}}</tspan>
            <tspan x="0" y="20%">{{knobs[1].name}}: {{knobs[1].value}}</tspan>
            <tspan x="0" y="30%" v-if="this.knobs[2]">{{knobs[2].name}}: {{knobs[2].value}}</tspan>
            <tspan x="0" y="40%" v-if="this.knobs[3]">{{knobs[3].name}}: {{knobs[3].value}}</tspan>
          </text>
          <text x="0" y="40%">
            <tspan x="0%" y="60%" v-if="this.knobs[4]">{{knobs[4].name}}: {{knobs[4].value}}</tspan>
            <tspan x="0%" y="70%" v-if="this.knobs[5]">{{knobs[5].name}}: {{knobs[5].value}}</tspan>
            <tspan x="0%" y="80%" v-if="this.knobs[6]">{{knobs[6].name}}: {{knobs[6].value}}</tspan>
            <tspan x="0%" y="90%" v-if="this.knobs[7]">{{knobs[7].name}}: {{knobs[7].value}}</tspan>
          </text>
        </g>
      </svg>
    </div>

</template>

<script>
import store from '../store' // path to your Vuex store

// TODO:
// [v]. translate the values so they work properly
// [v]. Gain
// [v]. turn the curve from the cubic one to quadratic
// [v]. make the lowpass & highpass one method
// [v]. do something about the colors

// [v] REDISiGN in accrodance with knobs !!
// [v] embed into modules
// [v] trying to move them back together

// ===============
// [v] the required margin data object and path
// [v] have the other knobs connected to the filter audio output
// [v] relativize and clean up the path drawing function (inc. the curve "global" variable)

// lfo:
// [v] create swing
// [v] have it move for sine - but it is hardcoded
// [x] have two other types
// [v] lfo prop watched/computed
// [v] transition
// [x] stylesheetapi
// http://danielcwilson.com/blog/2017/10/all-the-transform-ways/ - does not work, the properties are not recognized
// https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API
// [~] try transitions: define two kinds of transforms with class and switch between them
// [v] a bit more complicated : set interval with a callback function changing a bolean data property every given time, watch changes in the rate knob to call a function replacing the current interval time with a new one. Custom transition bound and used to indicate LFO's shape
// [v] cleanup
// STILL TODO:
// [] bug: the swing does not update when the knob is held/ slowly turned
// [] sawtooth timing

// Week 3.:
// circles no: the design way
// <circles> czy path?
// size changes circles move aparart
// dry/wet - size of the circles
//
//

export default {
  name: 'display',
  props: {
    module: {
      type: String
    },
    fill: {
      type: String
    },
    knobs: {
      type: Array,
      default () { return [] }
    },
    debug: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      // default values to not have the calculation derail at created()
      displayHeight: 300,
      displayWidth: 600,
      curveAmnt: 90,
      strokeWidth: '0.1',
      lfoRotation: 20,
      shouldItGoRight: false,
      shouldGoalGoRight: false,
      intervalId: null,
      goalIntervalId: null
    }
  },
  mounted () {
    console.log('display: mounted!', this.module)
    // update dimentions:
    this.updateDimensions()
    window.addEventListener('resize', this.updateDimensions())

    if (this.module == 'lfo') {
      var intervalTest = setInterval(this.updateSeconds, ((1 / this.knobs[3].value) * 1000))
      this.intervalId = intervalTest
      this.goalIntervalId = setInterval(this.updateGoalSeconds, (1 / (this.knobs[7].value)) * 1000)
    }
  },

  beforeDestroy () {
    window.removeEventListener('resize', this.updateDimensions())
  },
  methods: {
    updateSeconds () {
      this.shouldItGoRight = !this.shouldItGoRight
    },
    updateGoalSeconds () {
      this.shouldGoalGoRight = !this.shouldGoalGoRight
    },

    changeInterval (time) {
      clearInterval(this.intervalId)
      this.intervalId = setInterval(this.updateSeconds, time)
    },
    updateDimensions () {
      this.displayHeight = this.$refs.displayWrapper.clientHeight
      this.displayWidth = this.$refs.displayWrapper.clientWidth
    }
  },
  computed: {
    createModeIsActive () {
      return this.$store.state.gameState.createModeIsActive
    },
    // used to refer to LFO rate knob in a watcher (no way to do it directly using because of "[...]")
    firstCircleLeftMargin () {
      let margin = ''
      return margin
    },
    firstCircleLeftMarginGoal () {
      let margin = ''
      return margin
    },
    lfoRealFreq () {
      if (this.module === 'lfo') {
        return this.knobs[3].value
      }
    },
    path () {
      let line

      if (this.module === 'envelope') {
        // helpers:
        let fourthOfWidth = this.displayWidth / 4

        const attack = this.knobs[0]
        const decay = this.knobs[1]
        const sustain = this.knobs[2]
        const release = this.knobs[3]

        const attackXPosition = (attack.value / attack.max) * fourthOfWidth
        const attackYPosition = this.displayHeight * 0.75
        const decayXPosition = (decay.value / decay.max) * fourthOfWidth

        // the vertical decay position shall include a fix stopping it
        // from going all the way down (5% height) to perserve release indication:
        const decayYPosition = (1 - (sustain.value / sustain.max)) * (attackYPosition) - (1 - (sustain.value / sustain.max)) * (attackYPosition) * 0.05
        // no sustain, as it basically a horizontal line
        // release is known and shall be market with absolute position

        // svg path:
        line = 'M 0, ' + (this.displayHeight + 1) +
              ' l ' + attackXPosition + ', ' + (-attackYPosition) + ' ' +
              ' l ' + decayXPosition + ', ' + decayYPosition + ' ' +
              // a horizontal line representing sustain level including a fix regarding adding the release:
              ' h ' + (this.displayWidth - attackXPosition - decayXPosition - (((release.value / release.max)) * fourthOfWidth)) +
              // release end position:
              ' L ' + this.displayWidth + ', ' + (this.displayHeight + 1) + ' ' +
              ' Z'
      }

      if (this.module === 'filter') {
        // helpers:
        const type = this.knobs[0]
        const cutOffFreq = this.knobs[1]
        const q = this.knobs[2]
        const gain = this.knobs[3]

        let halfHeight = this.displayHeight / 2
        let halfWidth = this.displayWidth / 2
        const gainAddedDistance = ((gain.value / gain.max) * halfHeight) - 5
        const yOffset = 0
        const qDistance = (1 - (q.value / q.max)) * (halfWidth)
        const freqDistance = (cutOffFreq.value / cutOffFreq.max) * (halfWidth)

        // svg path:
        if (type.value === 'lowpass') {
          line = 'M 0,' + (this.displayHeight + 1) +
                ' v ' + (-(halfHeight + gainAddedDistance - yOffset)) +
                ' h ' + (freqDistance) +
                ' h ' + ((q.value / q.max) * (halfWidth) / 2) +
                ' q ' + (qDistance / 2) + ', 0 ' +
                      +qDistance + ', ' + (halfHeight + gainAddedDistance) +

                ' Z'
        } else if (type.value === 'highpass') {
          line = 'M 0,' + (this.displayHeight + 1) +
                ' h ' + freqDistance +
                ' h ' + ((q.value / q.max) * (halfWidth) / 2) +
                ' q ' + (qDistance / 2) + ', ' + (-(halfHeight + gainAddedDistance)) + ' ' +
                          qDistance + ', ' + (-(halfHeight + gainAddedDistance)) +
                ' h ' + this.displayWidth +
                ' v ' + this.displayHeight +
                ' Z'
        } else if (type.value === 'bandpass') {
          line = 'M 0, ' + (this.displayHeight + 1) +
                ' h ' + (freqDistance + (halfWidth / 2) - qDistance) +
                ' q ' + (qDistance / 2) + ', ' + (-(halfHeight + gainAddedDistance)) + ' ' +
                        qDistance + ', ' + (-(halfHeight + gainAddedDistance)) +
                ' q ' + (qDistance / 2) + ', 0 ' +
                        qDistance + ', ' + (halfHeight + gainAddedDistance) + ' ' +
                ' Z '
        }
      }

      if (this.module === 'oscillator') {
        // helpers:
        const octave = this.knobs[0]
        const detune = this.knobs[1]
        const phase = this.knobs[2]
        const type = this.knobs[3]

        const lineLength = 1.5 * this.displayWidth
        const yAxisMiddle = this.displayHeight / 2
        const h = yAxisMiddle / 2

        const octaveRatio = (octave.value / (octave.max - octave.min))
        const detuneRatio = ((detune.value / detune.max) * 1000) / (octave.max - octave.min)

        const iteration = 6.6 * h - h * (4.9 * octaveRatio) + (h * 0.3 * (1 - detuneRatio)) - 15

        let wave
        // square:
        if (type.value === 'square') {
          wave = ' v ' + h +
                 ' h ' + iteration +
                 ' v ' + (-yAxisMiddle) +
                 ' h ' + iteration +
                 ' v ' + h
        // sine:
        } else if (type.value === 'sine') {
          wave = ' q ' + '0, ' + h + ' ' + iteration / 2 + ', ' + h +
                 ' q ' + iteration / 2 + ', 0 ' + iteration / 2 + ', ' + (-h) +
                 ' q 0, ' + (-h) + ' ' + iteration / 2 + ' ' + (-h) +
                 ' q ' + iteration / 2 + ', 0 ' + ' ' + iteration / 2 + ' ' + h
        // sawtooth:
        } else if (type.value === 'sawtooth') {
          wave = ' v ' + h +
                 ' l ' + 2 * iteration + ', ' + (-yAxisMiddle) +
                 ' v ' + h
        // triangle
        } else if (type.value === 'triangle') {
          wave = ' l ' + iteration / 2 + ', ' + h +
                 ' l ' + iteration + ', ' + (-yAxisMiddle) +
                 ' l ' + iteration / 2 + ', ' + h
        }
        line = 'M ' + iteration * (phase.value / (phase.max)) + ', 0 ' +
              ' m ' + lineLength + ', ' + yAxisMiddle +
              ' h ' + (-lineLength - iteration) +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              ' Z '
      }

      if (this.module === 'lfo') {
        // NON OF THE BELOW PRESENETED APPROACHED WORKED (and should be deleted)
        // used a custom svg in the template

        // helpers:
        const r = this.displayHeight / 4
        const rate = this.knobs[0]
        const amount = this.knobs[1]
        const shape = this.knobs[2]

        // making a circle with a path..
        const circle = ' q ' + '0, ' + (-r) + ' ' + r + ', ' + (-r) +
                       ' q ' + r + ', 0 ' + r + ', ' + r +
                       ' q ' + '0, ' + r + ' ' + (-r) + ', ' + r +
                       ' q ' + (-r) + ', 0 ' + (-r) + ', ' + (-r)

        // the idea from https://codepen.io/jakob-e/pen/bgBegJ
        const archCrcl = 'M' + ((this.displayWidth / 2) - r) + ', ' + ((this.displayHeight / 2)) +
        // 'm '+(-r)+', 0'+
        'a ' + r + ', ' + r + ' 0 1,0 ' + (2 * r) + ',0' +
        'a ' + r + ', ' + r + ' 0 1,0 ' + (-2 * r) + ',0'

        line = ''
      }
      return line
    },

    lfoVB () {
      return (0 - this.displayWidth / 2) + ' 0 ' + this.displayWidth + ' ' + this.displayHeight
    },
    swingMovement () {
      // helpers:
      const rate = this.knobs[0]
      const amount = this.knobs[1]
      const shape = this.knobs[2]

      const rateRatio = (rate.value / (rate.max - rate.min))
      const amountRatio = (amount.value / (amount.max - amount.min))

      // a hot-fix to stop the swing from going trrrrrrrrr ---
      // let realFreq
      // if(this.knobs[3].value === 0) {
      //   realFreq = 0.1
      // } else {
      let realFreq = this.knobs[3].value
      // }

      const transitionTime = (1 / (realFreq))

      let transitionString
      // differenciate shapes using transitions:
      if (shape.value == 'sine') {
        transitionString = transitionTime + 's'
      } else if (shape.value == 'square') {
        transitionString = ''
      } else if (shape.value == 'sawtooth') {
        if (!this.shouldItGoRight) {
          transitionString = transitionTime + 's linear'
        } else {
          transitionString = ''
        }
      } else if (shape.value == 'triangle') {
        transitionString = transitionTime + 's linear'
      }
      let rotateAmount = 1900 * amountRatio
      let rotateString
      if (this.shouldItGoRight) {
        // go right:
        rotateString = 'rotate(' + rotateAmount + 'deg)'
      } else {
        // go left:
        rotateString = 'rotate(-' + rotateAmount + 'deg)'
      }
      return {transform: rotateString, transition: transitionString}
    },
    goalSwingMovement () {
      // helpers:
      const rate = this.knobs[4]
      const amount = this.knobs[5]
      const shape = this.knobs[6]

      const rateRatio = (rate.value / (rate.max - rate.min))
      const amountRatio = (amount.value / (amount.max - amount.min))

      const realGoalFreq = this.knobs[7].value

      const transitionTime = (1 / (realGoalFreq))

      let transitionString
      if (shape.value == 'sine') {
        transitionString = transitionTime + 's'
      } else if (shape.value == 'square') {
        transitionString = ''
      } else if (shape.value == 'sawtooth') {
        if (!this.shouldGoalGoRight) {
          transitionString = transitionTime + 's linear'
        } else {
          transitionString = ''
        }
      } else if (shape.value == 'triangle') {
        transitionString = transitionTime + 's linear'
      }
      let rotateAmount = 1900 * amountRatio
      let rotateString
      if (this.shouldGoalGoRight) {
        // go right:
        rotateString = 'rotate(' + rotateAmount + 'deg)'
      } else {
        // go left:
        rotateString = 'rotate(-' + rotateAmount + 'deg)'
      }
      return {transform: rotateString, transition: transitionString }
    },
    pathGoal () {
      let line

      if (this.module === 'lfo') {
        // helpers:
        const r = this.displayHeight / 4
        const rate = this.knobs[4]
        const amount = this.knobs[5]
        const shape = this.knobs[6]

        // making a circle with a path..
        const circle = ' q ' + '0, ' + (-r) + ' ' + r + ', ' + (-r) +
                       ' q ' + r + ', 0 ' + r + ', ' + r +
                       ' q ' + '0, ' + r + ' ' + (-r) + ', ' + r +
                       ' q ' + (-r) + ', 0 ' + (-r) + ', ' + (-r)

        // the idea from https://codepen.io/jakob-e/pen/bgBegJ
        const archCrcl = 'M' + ((this.displayWidth / 2) - r) + ', ' + ((this.displayHeight / 2)) +
        // 'm '+(-r)+', 0'+
        'a ' + r + ', ' + r + ' 0 1,0 ' + (2 * r) + ',0' +
        'a ' + r + ', ' + r + ' 0 1,0 ' + (-2 * r) + ',0'

        line = ''
      }

      if (this.module === 'envelope') {
        // helpers:
        let fourthOfWidth = this.displayWidth / 4

        const attack = this.knobs[4]
        const decay = this.knobs[5]
        const sustain = this.knobs[6]
        const release = this.knobs[7]

        const attackXPosition = (attack.value / attack.max) * fourthOfWidth
        const attackYPosition = this.displayHeight * 0.75
        const decayXPosition = (decay.value / decay.max) * fourthOfWidth

        // the vertical decay position shall include a fix stopping it
        // from going all the way down (5% height) to perserve release indication:
        const decayYPosition = (1 - (sustain.value / sustain.max)) * (attackYPosition) - (1 - (sustain.value / sustain.max)) * (attackYPosition) * 0.05
        // no sustain, as it basically a horizontal line
        // release is known and shall be market with absolute position

        // svg path:
        line = 'M 0, ' + (this.displayHeight + 1) +
              ' l ' + attackXPosition + ', ' + (-attackYPosition) + ' ' +
              ' l ' + decayXPosition + ', ' + decayYPosition + ' ' +
              // a horizontal line representing sustain level including a fix regarding adding the release:
              ' h ' + (this.displayWidth - attackXPosition - decayXPosition - (((release.value / release.max)) * fourthOfWidth)) +
              // release end position:
              ' L ' + this.displayWidth + ', ' + (this.displayHeight + 1) + ' '
      }

      if (this.module === 'filter') {
        // helpers:
        const type = this.knobs[4]
        const cutOffFreq = this.knobs[5]
        const q = this.knobs[6]
        const gain = this.knobs[7]

        let halfHeight = this.displayHeight / 2
        let halfWidth = this.displayWidth / 2
        const gainAddedDistance = ((gain.value / gain.max) * halfHeight) - 5
        const yOffset = 0
        const qDistance = (1 - (q.value / q.max)) * (halfWidth)
        const freqDistance = (cutOffFreq.value / cutOffFreq.max) * (halfWidth)

        // svg path:
        if (type.value === 'lowpass') {
          line = 'M 0,' + ((this.displayHeight + 1) - (halfHeight + gainAddedDistance - yOffset)) +
                // ' v ' + (-(halfHeight + gainAddedDistance - yOffset)) +
                ' h ' + (freqDistance) +
                ' h ' + ((q.value / q.max) * (halfWidth) / 2) +
                ' q ' + (qDistance / 2) + ', 0 ' +
                      +qDistance + ', ' + (halfHeight + gainAddedDistance)
        } else if (type.value === 'highpass') {
          line = 'M' + (freqDistance + ((q.value / q.max) * (halfWidth) / 2)) + ', ' + (this.displayHeight + 1) +
                ' q ' + (qDistance / 2) + ', ' + (-(halfHeight + gainAddedDistance)) + ' ' +
                          qDistance + ', ' + (-(halfHeight + gainAddedDistance)) +
                ' h ' + this.displayWidth +
                ' v ' + this.displayHeight
        } else if (type.value === 'bandpass') {
          line = 'M' + (freqDistance + (halfWidth / 2) - qDistance) + ', ' + (this.displayHeight + 1) +
                ' q ' + (qDistance / 2) + ', ' + (-(halfHeight + gainAddedDistance)) + ' ' +
                        qDistance + ', ' + (-(halfHeight + gainAddedDistance)) +
                ' q ' + (qDistance / 2) + ', 0 ' +
                        qDistance + ', ' + (halfHeight + gainAddedDistance) + ' '
        }
      }

      if (this.module === 'oscillator') {
        // helpers:
        const octave = this.knobs[4]
        const detune = this.knobs[5]
        const phase = this.knobs[6]
        const type = this.knobs[7]

        const lineLength = 1.5 * this.displayWidth
        const yAxisMiddle = this.displayHeight / 2
        const h = yAxisMiddle / 2

        const octaveRatio = (octave.value / (octave.max - octave.min))
        const detuneRatio = ((detune.value / detune.max) * 1000) / (octave.max - octave.min)

        const iteration = 6.6 * h - h * (4.9 * octaveRatio) + (h * 0.3 * (1 - detuneRatio)) - 15

        let wave
        // square:
        if (type.value === 'square') {
          wave = ' v ' + h +
                 ' h ' + iteration +
                 ' v ' + (-yAxisMiddle) +
                 ' h ' + iteration +
                 ' v ' + h
        // sine:
        } else if (type.value === 'sine') {
          wave =
                ' q ' + '0, ' + h + ' ' + iteration / 2 + ', ' + h +
                 ' q ' + iteration / 2 + ', 0 ' + iteration / 2 + ', ' + (-h) +
                 ' q 0, ' + (-h) + ' ' + iteration / 2 + ' ' + (-h) +
                 ' q ' + iteration / 2 + ', 0 ' + ' ' + iteration / 2 + ' ' + h
        // sawtooth:
        } else if (type.value === 'sawtooth') {
          wave = ' v ' + h +
                 ' l ' + 2 * iteration + ', ' + (-yAxisMiddle) +
                 ' v ' + h
        // triangle
        } else if (type.value === 'triangle') {
          wave = ' l ' + iteration / 2 + ', ' + h +
                 ' l ' + iteration + ', ' + (-yAxisMiddle) +
                 ' l ' + iteration / 2 + ', ' + h
        }
        line =
              ' m -' + iteration + ', ' + yAxisMiddle +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave +
              wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave + wave
      }

      return line
    },
    centerLFOSwing () {
      return 'transform: translateX(' + (this.displayWidth / 2) + 'px)'
    }
    // moveLFOSwingUp() {
    //   return 'translateY(-' + (this.displayHeight/2) + 'px)'
    // }
  },
  watch: {
    // used to update the interval length on rate knob turn
    lfoRealFreq: {
      handler (newValue, oldValue) {
        if (this.module == 'lfo') {
          let realFreq = this.knobs[3]
          this.changeInterval((1 / (newValue)) * 1000)
        }
      },
      deep: true
    }

  }
}
</script>

<style scoped lang="scss">
  path {
    display: inline-block;
  }
  .swingClass {
    transform-origin: 0% -40%;
  }
  .debug-text {
    font: bold 20px sans-serif;
    fill: red;
  }
</style>
