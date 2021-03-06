<template>
  <div class="game">
    <game-nav-bar/>
    <div class="level">
      <oscillator-module-one
        v-if="moduleIsUseable('oscillator1')"
        :class="[(activeModule == 0 ? 'active' : '')]"
      />
      <oscillator-module-two
        v-if="moduleIsUseable('oscillator2')"
        :class="[(activeModule == 1 ? 'active' : '')]"
      />
      <filter-module
        v-if="moduleIsUseable('filter')"
        :class="[(activeModule == 2 ? 'active' : '')]"
      />
      <envelope-module
        v-if="moduleIsUseable('envelope')"
        :class="[(activeModule == 3 ? 'active' : '')]"
      />
      <lfo-module
        v-if="moduleIsUseable('lfo')"
        :class="[(activeModule == 4 ? 'active' : '')]"
      />
      <sequencer-module
        v-if="createModeIsActive"
        class="module sequencer"
      />
    </div>
    <div class="tabs">
      <div @click="showOsc1" v-if="moduleIsUseable('oscillator1')" class="tabs__tab tabs__osc"><span>Osc 1</span><span
          :class="{
                    'module__name__status-indicator': true,
                    'module__name__status-indicator--active': oscillator1Complete
                  }"
          :style="{
                    'background-color': oscillator1Complete ? oscillatorColor : '',
                    'box-shadow': oscillator1Complete ? `0px 0px 16px ${oscillatorColor}` : '',
                  }"
        ></span></div>
      <div @click="showOsc2" v-if="moduleIsUseable('oscillator2')" class="tabs__tab tabs__osc"><span>Osc 2</span><span
          v-if="moduleIsUseable('oscillator2')"
          :class="{
                    'module__name__status-indicator': true,
                    'module__name__status-indicator--active': oscillator2Complete
                  }"
          :style="{
                    'background-color': oscillator1Complete ? oscillatorColor : '',
                    'box-shadow': oscillator1Complete ? `0px 0px 16px ${oscillatorTwoColor}` : '',
                  }"
        ></span></div>
      <div @click="showFil" v-if="moduleIsUseable('filter')" class="tabs__tab tabs__filter"><span>Filter</span><span
        v-if="moduleIsUseable('filter')"
        :class="{
                  'module__name__status-indicator indicator__osctwo': true,
                  'module__name__status-indicator--active indicator__osctwo': filterComplete
                }"
            :style="{
                  'background-color': filterComplete ? filterColor : '',
                  'box-shadow': filterComplete ? `0px 0px 16px ${filterColor}` : '',
                }"
      ></span></div>
      <div @click="showEnv" v-if="moduleIsUseable('envelope')" class="tabs__tab tabs__env"><span>Env</span><span
        v-if="moduleIsUseable('envelope')"
        :class="{
                  'module__name__status-indicator indicator__envelope': true,
                  'module__name__status-indicator--active indicator__envelope': envelopeComplete
                }"
        :style="{
                  'background-color': envelopeComplete ? envelopeColor : '',
                  'box-shadow': envelopeComplete ? `0px 0px 16px ${envelopeColor}` : '',
                }"
      ></span></div>
      <div @click="showLfo" v-if="moduleIsUseable('lfo')" class="tabs__tab tabs__lfo"><span>Lfo</span><span
        v-if="moduleIsUseable('lfo')"
        :class="{
                  'module__name__status-indicator indicator__lfo': true,
                  'module__name__status-indicator--active indicator__lfo': lfoComplete
                }"
        :style="{
                  'background-color': lfoComplete ? lfoColor : '',
                  'box-shadow': lfoComplete ? `0px 0px 16px ${lfoColor}` : '',
                }"
      ></span></div>
    </div>
  </div>
</template>

<script>
import { MODULE_OSCILLATOR_COLOR, MODULE_OSCILLATORTWO_COLOR, MODULE_ENVELOPE_COLOR, MODULE_FILTER_COLOR, MODULE_DELAY_COLOR, MODULE_REVERB_COLOR, MODULE_LFO_COLOR} from '@/constants'
import some from 'lodash/some'
// @ is an alias to /src
import GameNavBar from '@/components/GameNavBar.vue'
import OscillatorModuleOne from '@/components/module/OscillatorModuleOne.vue'
import OscillatorModuleTwo from '@/components/module/OscillatorModuleTwo.vue'
import EnvelopeModule from '@/components/module/EnvelopeModule.vue'
import FilterModule from '@/components/module/FilterModule.vue'
import LfoModule from '@/components/module/LfoModule.vue'
import SequencerModule from '@/components/module/SequencerModule.vue'

export default {
  name: 'home',
  data () {
    return {
      activeModule: 0,
      marginArray: [0, 0.2, 0.4, 0.6],
      indicatorActive: true,
      oscillatorColor: MODULE_OSCILLATOR_COLOR,
      oscillatorTwoColor: MODULE_OSCILLATORTWO_COLOR,
      envelopeColor: MODULE_ENVELOPE_COLOR,
      filterColor: MODULE_FILTER_COLOR,
      lfoColor: MODULE_LFO_COLOR
    }
  },
  components: {
    GameNavBar,
    OscillatorModuleOne,
    OscillatorModuleTwo,
    EnvelopeModule,
    FilterModule,
    LfoModule,
    SequencerModule
  },
  methods: {
    moduleIsUseable (moduleName) {
      if (this.createModeIsActive) return true
      return some(this.knobsAvailable[moduleName]) // some are truthy
    },
    showOsc1 () {
      this.activeModule = 0
    },
    showOsc2 () {
      this.activeModule = 1
    },
    showFil () {
      this.activeModule = 2
    },
    showEnv () {
      this.activeModule = 3
    },
    showLfo () {
      this.activeModule = 4
    },
    moduleIsUseable (moduleName) {
      if (this.createModeIsActive) return true
      return some(this.knobsAvailable[moduleName]) // some are truthy
    }
  },
  computed: {
    createModeIsActive () {
      return this.$store.state.gameState.createModeIsActive
    },
    knobsAvailable () {
      return this.$store.state.gameState.knobsAvailable
    },
    knobsAvailable () {
      return this.$store.state.gameState.knobsAvailable
    },
    oscillator1Complete () {
      return Object.values(this.$store.getters.audioParametersMatchGoalWithMargin['oscillator1']).every(param => param)
    },
    oscillator2Complete () {
      return Object.values(this.$store.getters.audioParametersMatchGoalWithMargin['oscillator2']).every(param => param)
    },
    filterComplete () {
      return Object.values(this.$store.getters.audioParametersMatchGoalWithMargin['filter']).every(param => param)
    },
    envelopeComplete () {
      return Object.values(this.$store.getters.audioParametersMatchGoalWithMargin['envelope']).every(param => param)
    },
    lfoComplete () {
      return Object.values(this.$store.getters.audioParametersMatchGoalWithMargin['lfo']).every(param => param)
    }
  }
}
</script>

<style scoped lang="scss">

.module__name__status-indicator {
  display: inline-block;
  width: 11px;
  height: 11px;
  border-radius: 100%;
  transition: 0.5s;
 border: 1px solid white;

  &--active {
    border: none;
    transition: 0.5s;
 border: 1px solid white;

  }
}

.active {
    left: 0;
}

</style>
