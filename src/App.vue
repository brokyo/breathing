<template>
<div id="app">
  <div id="introCopy">
    <div id="pageHeader">
      <span>This is the full version of the breathing exercise from the <a href="http://www.digitalprayers.work" target="_blank"><span class="prayersLink">Digital Prayers</span></a> desktop application.</span>
    </div>
    <div id="pageIntro">
      <p>If you'd like to see it in context please download the desktop app (strongly recommended) [note: you're here early and this doesn't yet exist] or visit the site using the chrome browser.</p>
      <p>Otherwise after clicking `start` the drone will evolve on its own but you can override. Feel free to <a href="mailto:alexcarusillo@gmail.com">email me</a> if you make something interesting.</p>
    </div>
    <span id="startButton"
          v-if="!droneStarted"
          @click="startDrone(); droneStarted = true">Click To Start Drone</span>
  </div>
  <div id="sketch">
    <div id="sketch-holder"></div>
    <span id="instruction"
          v-if="showInstruction">{{currentInstruction}}</span>
  </div>
  <div id="noiseControls"
       v-show="droneStarted">
    <div class="individualLine"
         v-for="(lineConfig, index) in droneConfig.noise">
      <span class="title">Noise Line {{index}}</span>
      <div>
        <div v-for="config in droneConfig.filterConfig"
             v-if="config.id == index">
          <label>Chord Base [each note transposed up and down one ocave]</label>
          {{config.chord}}
          <select v-model="config.chord"
                  @change="changeChord($event.target.value, index)">
            <option v-for="chord in chords">{{chord}}</option>
          </select>
          <label>Filter Q [amount of white noise]</label>
          <select v-model="config.Q"
                  @change="changeFilterQ(config.Q, index)">
            <option v-for="q in qOptions">{{q}}</option>
          </select>
        </div>
        <div v-for="config in droneConfig.originPannerConfig"              
              v-if="config.id == index">
         <label>Noise Base Location [Pan]</label>
         <input type="range"
                min="-1"
                max="1"
                step="0.05"
                v-model="config.pan"
                @change="changePan($event.target.value, index)"
                >
        </div>
        <label>Playback Rate</label>
        <input type="range"
               min="0.1"
               max="0.9"
               step="0.05"
               v-model="lineConfig.playbackRate"
               @change="changeNoisePlaybackRate(lineConfig.playbackRate, index)" />
        <label>Volume</label>
        <input type="range"
               min="-25"
               max="0"
               step="1"
               v-model="lineConfig.volume"
               @change="changeNoiseVolume(lineConfig.volume, index)" />
      </div>
    </div>
  </div>
  <div id="roomControlsContainer"
       v-show="droneStarted">
    <span class="droneControls">Room Texture Controls [In Order Of Effect]</span>
    <div id="controls">
      <div id="feedbackDelay"
           class="patchConfig">
        <div class="title">
          <label>feedback delay</label>
        </div>
        <div class="config">
          <label>Audibility</label>
          <input type="range"
                 v-model.number="droneConfig.feedbackDelayConfig.wet"
                 min="0"
                 max="1"
                 step="0.05"></input>
          <label>Delay Time</label>
          <input type="range"
                 v-model.number="droneConfig.feedbackDelayConfig.delayTime"
                 min="0"
                 max="5"
                 step="0.25"></input>
          <label>Feedback</label>
          <input type="range"
                 v-model.number="droneConfig.feedbackDelayConfig.feedback"
                 min="0"
                 max="1"
                 step="0.05"></input>
        </div>
        <div class="code">
          <pre>{{droneConfig.feedbackDelayConfig}}</pre>
        </div>
      </div>
      <div id="autoPanner"
           class="patchConfig">
        <div class="title">
          <label>auto panner</label>
        </div>
        <div class="config">
          <label>Frequency</label>
          <input type="range"
                 v-model.number="droneConfig.autoPannerConfig.frequency"
                 min="0"
                 max="1"
                 step="0.05"></input>
          <label>Oscillator Type</label>
          <select v-model="droneConfig.autoPannerConfig.type">
            <option v-for="oscillator in oscillatorArray">{{oscillator}}</option>
          </select>
          <label>Depth</label>
          <input type="range"
                 v-model.number="droneConfig.autoPannerConfig.depth"
                 min="0"
                 max="1"
                 step="0.05"></input>
        </div>
        <div class="code">
          <pre>{{droneConfig.autoPannerConfig}}</pre>
        </div>
      </div>
      <div id="reverb"
           class="patchConfig">
        <div class="title">
          <label>reverb</label>
        </div>
        <div class="config">
          <label>Room Size</label>
          <input type="range"
                 v-model.number="droneConfig.freeverbConfig.roomSize"
                 min="0"
                 max="1"
                 step="0.05"></input>
          <label>Dampening</label>
          <input type="range"
                 v-model.number="droneConfig.freeverbConfig.dampening"
                 min="0"
                 max="5000"
                 step="10"></input>
        </div>
        <div class="code">
          <pre>{{droneConfig.freeverbConfig}}</pre>
        </div>
      </div>
      <div id="chorus"
           class="patchConfig">
        <div class="title">
          <label>chorus</label>
        </div>
        <div class="config">
          <label>Audibility</label>
          <input type="range"
                 v-model.number="droneConfig.chorusConfig.wet"
                 min="0"
                 max="1"
                 step="0.05"></input>
          <label>Frequency</label>
          <input type="range"
                 v-model.number="droneConfig.chorusConfig.frequency"
                 min="0.1"
                 max="3"
                 step="0.025"></input>
          <label>Delay Time</label>
          <input type="range"
                 v-model.number="droneConfig.chorusConfig.delayTime"
                 min="0"
                 max="5"
                 step="0.1"></input>
          <label>Depth</label>
          <input type="range"
                 v-model.number="droneConfig.chorusConfig.depth"
                 min="0"
                 max="1"
                 step="0.05"></input>
          <label>Oscillator Type</label>
          <select v-model="droneConfig.chorusConfig.type">
            <option v-for="oscillator in oscillatorArray">{{oscillator}}</option>
          </select>
          <label>Spread</label>
          <input type="range"
                 v-model.number="droneConfig.chorusConfig.spread"
                 min="0"
                 max="180"
                 step="1"></input>
        </div>
        <div class="code">
          <pre>{{droneConfig.chorusConfig}}</pre>
        </div>
      </div>
    </div>
  </div>
</div>

</template>

<script>
import p5 from 'p5';
import Tone from 'tone';
import _ from 'lodash';

function randItem(arr) {
  return arr[Math.floor(Math.random() * arr.length)]
}

// TODO: Leaving in this in pure js as I think there's a performance hit? Ask somebody about that....
// p5 setup
var xoff = 0
var yoff = 0
var zoff = 0
var rOffset = 20
var gOffset = 0
var bOffset = 20
var counter = 0
var rInc = 0
var gInc = 0
var bInc = 0
var rIncBase = 1
var gIncBase = 0
var bIncBase = 0.5

export default {
  data() {
    return {
      // meta
      droneStarted: true,
      showExplanation: false,

      // p5
      showInstruction: true,
      currentInstruction: 'in...',
      xinc: 0.009,
      yinc: 0.007,
      // zinc: 0.0000015,
      zinc: 0.000015,

      // Tone
      oscillatorArray: [
        'sine',
        'square',
        'sawtooth',
        'triangle'
      ],
      qOptions: [
        400,
        500,
        600,
        800,
        1200,
        1600,
        2000
      ],
      chords: [
          [
            'C3',
            'E3',
            'G3',
            'B3',
            'C3'
          ],
          [
            'A3',
            'C4',
            'E4',
            'G4',
            'A3'
          ],
          [
            'D3',
            'G3',
            'A3',
            'C4',
            'E4'
          ],
          [
            'E3',
            'G3',
            'B3',
            'D4',
            'E3'
          ]
      ],
      droneConfig: {
        noise: [
          {
            type: 'brown',
            volume: -10,
            playbackRate: 0.4
          },
          {
            type: 'brown',
            fadeIn: -18,
            playbackRate: 0.2
          }
        ],
        filterConfig: {
          line0: {
            id: 0,
            chord: [],
            Q: null
          },
          line1: {
            id: 1,
            chord: [],
            Q: null
          }
        },
        originPannerConfig: {
          line0: {
            id: 0,
            min: -0.7,
            max: -0.1,
            pan: -0.2
          },
          line1: {
            id: 1,
            min: 0.7,
            max: 0.1,
            pan: 0.2
          }
        },
        feedbackDelayConfig: {
          feedback: 0.45,
          delayTime: 0.75,
          maxDelay: 5,
          wet: 1
        },
        autoPannerConfig: {
          frequency: 0.35,
          type: 'square',
          depth: 0.3
        },
        freeverbConfig: {
          roomSize: 0.9,
          dampening: 2000
        },
        chorusConfig: {
          frequency: 0.35,
          delayTime: 3.5,
          depth: 0.7,
          type: 'sine',
          spread: 180
        },
        compressorConfig: {
          ratio: 12,
          threshold: -24
        }
      },
      audioLine: {
        noise: {
          line0: {},
          line1: {}
        },
        filters: {
          line0: [],
          line1: []
        },
        panners: {
          line0: {},
          line1: {}
        },
        in: {},
        delay: {},
        panner: {},
        reverb: {},
        chorus: {},
        gain: {},
        compressor: {}
      }
    }
  },
  watch: {
    'droneConfig.feedbackDelayConfig': {
      handler: function() {
        this.audioLine.delay.set(this.droneConfig.feedbackDelayConfig)
      },
      deep: true
    },
    'droneConfig.autoPannerConfig': {
      handler: function() {
        this.audioLine.panner.set(this.droneConfig.autoPannerConfig)
      },
      deep: true
    },
    'droneConfig.freeverbConfig': {
      handler: function() {
        this.audioLine.reverb.set(this.droneConfig.freeverbConfig)
      },
      deep: true
    },
    'droneConfig.chorusConfig': {
      handler: function() {
        this.audioLine.chorus.set(this.droneConfig.chorusConfig)
      },
      deep: true
    }
  },
  methods: {
    // p5
    breathPause(direction) {
      rInc = 0
      bInc = 0
      gInc = 0
      this.currentInstruction = 'pause'
      if (direction === 'down') {
        setTimeout(this.colorDown, 1500)
      } else {
        setTimeout(this.colorUp, 1500)
      }
    },
    colorUp() {
      this.currentInstruction = 'in...'
      rInc = rIncBase
      bInc = bIncBase
      gInc = gIncBase
      setTimeout(this.breathPause, 4000, 'down')
    },
    colorDown() {
      this.currentInstruction = 'out...'
      rInc = -(rIncBase / 2)
      bInc = -(bIncBase / 2)
      gInc = -(gIncBase / 2)
      setTimeout(this.breathPause, 8000, 'up')
    },

    // Tone - Texture
    setupTone() {
      this.audioLine.in = new Tone.Gain()
      this.audioLine.delay = new Tone.FeedbackDelay(this.droneConfig.feedbackDelayConfig)
      this.audioLine.panner = new Tone.AutoPanner(this.droneConfig.autoPannerConfig).start()
      this.audioLine.reverb = new Tone.Freeverb(this.droneConfig.freeverbConfig)
      this.audioLine.chorus = new Tone.Chorus(this.droneConfig.chorusConfig)
      this.audioLine.gain = new Tone.Gain(0.75)
      this.audioLine.compressor = new Tone.Compressor(this.droneConfig.compressorConfig)

    },
    startDrone() {
      this.colorUp()

      // Notes are just bandpass filters on white noise. 
      // Allows for texture and less intensive note/effect changes

      // Create the white noise source
      this.audioLine.noise.line0 = new Tone.Noise({
        type: 'brown',
        fadeIn: 3,
        playbackRate: 0.4
      }).start()

      this.audioLine.noise.line1 = new Tone.Noise({
        type: 'brown',
        fadeIn: 3,
        playbackRate: 0.4
      })

      this.audioLine.panners.line0 = new Tone.Panner(-0.2)
      this.audioLine.panners.line1 = new Tone.Panner(-0.2)

      // Select random chord from list
      let newChord = _.sampleSize(this.chords, 2)
      let newQ = _.sampleSize(this.qOptions, 2)

      // Take each note, transpose it up and down one octave, 
      // and create a filter letting those frequencies pass through
      newChord.map((chord, index) => {
        let lineName = 'line' + String(index)
        this.droneConfig.filterConfig[lineName].chord = newChord[index]
        this.droneConfig.filterConfig[lineName].Q = newQ[index]
        chord.map(note => {

          // convert note to tone object
          let frequency = new Tone.Frequency(note)

          // transpose and create bandpass filter
          let transposedFrequencies = [
            frequency,
            frequency.transpose(+12),
            frequency.transpose(-12)
          ].map(transposedFreq => {
            let filter = new Tone.Filter({
              type: 'bandpass',
              frequency: transposedFreq,
              Q: newQ[index]
            })

            // connect the filter and store a reference to it so it can be changed
            let lineParam = 'line' + String(index)
            this.audioLine.noise[lineParam].connect(filter)
            filter.connect(this.audioLine.panners[lineParam])
            this.audioLine.panners[lineParam].connect(this.audioLine.in)

            this.audioLine.filters[lineParam].push(filter)

          })

        })
      })

      setTimeout(this.changeNoiseParams, 25000, this)
    },
    changePan(location, index) {
      var lineName = 'line' + String(index)
      console.log('New Pan @', lineName, '| Value:', location)

      this.droneConfig.originPannerConfig[lineName].pan = location;
      this.audioLine.panners[lineName].set({"pan": location})
    },
    changeChord(chord, index) {
      let lineName = 'line' + String(index)
      let chordBase = JSON.parse(chord)

      this.droneConfig.filterConfig[lineName].chord = chordBase
      console.log('New Chord @', lineName, '| Value:', chordBase)

      chordBase.map((note, noteIndex) => {
          // convert note to tone object
          let frequency = new Tone.Frequency(note)

          // transpose and create bandpass filter
          let transposedFrequencies = [
            frequency,
            frequency.transpose(+12),
            frequency.transpose(-12)
          ].map((transposedFreq, transposedIndex) => {
            let filterIndex = (noteIndex * 3) + transposedIndex
            this.audioLine.filters[lineName][filterIndex].set({
              frequency: transposedFreq
            })
          })
      })
    },
    changeFilterQ(newQ, index) {
      var lineName = 'line' + String(index)
      console.log('New Q @', lineName, '| Value:', newQ)

      this.droneConfig.filterConfig[lineName].Q = newQ
      this.audioLine.filters[lineName].map(filter => {
        filter.Q.rampTo(newQ, 5)
      })
    },
    changeNoiseVolume(newVolume, index) {
      let lineName = 'line' + String(index)
      console.log('New Volume @', lineName, '| Value:', newVolume)

      this.droneConfig.noise[index].volume = newVolume
      this.audioLine.noise[lineName].volume.linearRampTo(newVolume, 20)
    },
    changeNoisePlaybackRate(newRate, index) {
      let lineName = 'line' + String(index)
      console.log('New Rate @', lineName, '| Value:', newRate)

      this.droneConfig.noise[index].playbackRate = newRate
      this.audioLine.noise[lineName].set({
        'playbackRate': newRate
      })
    },
    changeNoiseParams(vue) {
      let affectedLine = _.sample([
        0,
        1
      ])
      let changeIndex = _.random(1, 5)
      changeIndex = 5

      if (changeIndex === 1) {
        // Dumb hack so random matches the value the html selector. Surely there's a better way to do this.
        let chordHack = JSON.stringify(_.sample(this.chords))
        vue.changeChord(chordHack, affectedLine)
      } else if (changeIndex === 2) {
        let nextQ = _.sample(vue.qOptions)
        vue.changeFilterQ(nextQ, affectedLine)
      } else if (changeIndex === 3) {
        let nextVol = _.random(-20, -5)
        vue.changeNoiseVolume(nextVol, affectedLine)
      } else if (changeIndex === 4) {
        let nextPlaybackRate = _.random(0.1, 0.8)
        vue.changeNoisePlaybackRate(nextPlaybackRate, affectedLine)
      } else if (changeIndex === 5) {
        let newLocation = _.random(0.1, 0.7)
        if (affectedLine === 0){
          let newLocation = -newLocation
        }
        vue.changePan(newLocation, affectedLine)
      }

      setTimeout(vue.changeNoiseParams, _.random(20000, 40000), vue)
    }
  },
  mounted() {
    this.setupTone()

    // Setup Sketch
    var viz = new p5(sketch => {
      sketch.setup = () => {
        var canvas = sketch.createCanvas(800, 450)
        canvas.parent('sketch-holder')
        sketch.pixelDensity(1)
        this.droneStarted = false
      // sketch.frameRate(5)
      }
      sketch.draw = () => {
        if (this.droneStarted) {
          sketch.background(51)

          zoff = 0

          sketch.loadPixels()
          for (var x = 0; x < sketch.width; x++) {
            xoff = 0

            for (var y = 0; y < sketch.height; y++) {

              var index = (x + y * sketch.width) * 4
              var r = sketch.noise(xoff, yoff, zoff) * 255
              sketch.pixels[index + 0] = r + rOffset
              sketch.pixels[index + 1] = r + gOffset
              sketch.pixels[index + 2] = r + bOffset

              xoff += Number(this.xinc)
              zoff += Number(this.zinc)
            }
          }
          sketch.updatePixels()
          yoff += Number(this.yinc)
          rOffset += rInc
          gOffset += gInc
          bOffset += bInc
        }
      }
    })

    // Connect Tone Line
    this.audioLine.in.connect(this.audioLine.delay)
    this.audioLine.delay.connect(this.audioLine.panner)
    this.audioLine.panner.connect(this.audioLine.reverb)
    this.audioLine.reverb.connect(this.audioLine.chorus)
    this.audioLine.chorus.connect(this.audioLine.gain)
    this.audioLine.gain.connect(this.audioLine.compressor)
    this.audioLine.compressor.connect(Tone.Master)
  }
};

</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Chakra+Petch|Roboto');
#app {
  font-family: 'Roboto', sans-serif;
}

#pageHeader {
  font-family: 'Chakra Petch', sans-serif;
  font-size: 40px;
  width: 1000px;
}

#pageIntro {
  margin-top: 30px;
  font-size: 16px;
  margin-bottom: 30px;
}

#startButton {
  cursor: pointer;
  background: white;
  padding: 10px;
  font-size: 22px;
  border: 2px solid black;
  width: 200px;
}

#startButton:hover {
  background: #D2E9FC;
}

#sketch {
  width: 800px;
  position: relative;
  #instruction {
    font-family: 'Chakra Petch', sans-serif;
    position: absolute;
    bottom: 10px;
    right: 10px;
    color: white;
  }
}

#noiseControls {
  display: flex;
  font-family: 'Chakra Petch', sans-serif;
  margin-top: 40px;
  .individualLine {
    flex-grow: 1;
    align-content: center;
    .title {
      font-size: 22px;
      text-decoration: underline;
    }
    label {
      display: block;
      margin-top: 10px;
    }
    input[type='range'] {
      width: 80%;
    }
  }
}

#roomControlsContainer {
  margin-top: 40px;
  font-family: 'Chakra Petch', sans-serif;
  .droneControls {
    display: block;
    font-size: 22px;
    text-decoration: underline;
    margin-bottom: 15px;
  }
  .selectControl {
    margin-bottom: 5px;
    label {
      margin-right: 10px;
    }
  }
}

.patchConfig {
  width: 100%;
  margin-bottom: 20px;
  display: flex;
  border: 1px solid #7E7E7E;
  #partialConfig {
    display: flex;
    width: 100%;
  }
  .title {
    display: flex;
    position: relative;
    flex-grow: 1;
    background-color: #ba9eb49e;
    flex-basis: 0;
    justify-content: center;
    label {
      font-family: sans-serif;
      justify-content: center;
      margin-top: 10px;
      text-align: center;
      font-weight: 500;
    }
    #buttons {
      position: absolute;
      top: 40px;
    }
    #filterButton {
      position: absolute;
      top: 40px;
      width: 80%;
    }
  }
  .config {
    padding: 10px;
    flex-basis: 0;
    flex-grow: 7;
    select {
      margin: 2px;
    }
  }
  .code {
    background-color: #D7D7D7;
    color: #484848;
    font-size: 11px;
    text-align: left;
    font-weight: 500;
    flex-basis: 0;
    flex-grow: 2;
  }
  label {
    display: block;
    font-weight: 500;
  }
  input[type="range"] {
    display: block;
    width: 100%;
  }
}
</style>
