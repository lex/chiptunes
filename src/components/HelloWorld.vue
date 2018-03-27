<template>
  <div>
    <h1>{{ title }}</h1>
    <div class="tracker-container">
      <div class="np-container">
        <div v-for="i in channelLength" :key="`header-${i - 1}`" :class="`column-header column-header-${i - 1 === currentPosition ? 'active' : 'inactive'}`">{{`${(i &lt;= 16) ? '0' : ''}${(i - 1).toString(16).toUpperCase()}`}}</div>
      </div>
      <div class="tracks-container">
        <div class="track-container" v-for="(channel, channelIndex) in channels" :key="channelIndex">
          <div v-for="(note, i) in channel" :class="`grid-item grid-item-${note.selected ? 'selected' : 'unselected'}`" :key="i" :id="`${note.channel}:${note.index}`" v-on:click="clicked">{{note.selected ? note.note : ''}}</div>
        </div>
      </div>
      <div>
        <input type="radio" id="sine" value="sine" v-model="selectedWaveform">
        <label for="sine">Sine</label>
        <br>
        <input type="radio" id="square" value="square" v-model="selectedWaveform">
        <label for="square">Square</label>
        <br>
        <input type="radio" id="triangle" value="triangle" v-model="selectedWaveform">
        <label for="triangle">Triangle</label>
        <br>
        <input type="radio" id="sawtooth" value="sawtooth" v-model="selectedWaveform">
        <label for="sawtooth">Sawtooth</label>
        <br>
        <input type="radio" id="snare" value="snare" v-model="selectedWaveform">
        <label for="snare">Snare</label>
        <br>
        <input type="text" id="note" v-model="note">
        <label for="note">Note</label>
        <br>
        <input type="text" id="noteLength" v-model="noteLength">
        <label for="noteLength">Length</label>
        <br>
        <input type="text" id="json" v-model="songAsJson">
        <label for="json">JSON</label>
        <br>
        <button v-on:click="play">Play</button>
        <br>
        <button v-on:click="stop">Stop</button>
      </div>
    </div>
  </div>
</template>

<script>
import Tone from 'tone';

const initialChannelLength = 20;
const waveforms = ['sine', 'square', 'triangle', 'sawtooth'];

export default {
  name: 'HelloWorld',
  data() {
    return {
      title: 'Chiptunes',
      channelLength: initialChannelLength,
      currentPosition: -1,
      tileColor: 'orange',
      channels: [
        this.createChannel(0, initialChannelLength),
        this.createChannel(1, initialChannelLength),
        this.createChannel(2, initialChannelLength),
      ],
      selectedWaveform: 'sine',
      note: 'C4',
      noteLength: '4n',
      loop: null,
      channelSynths: null,
    };
  },
  methods: {
    clicked(e) {
      const id = e.target.id;
      const split = id.split(':');
      const channel = split[0];
      const slot = split[1];
      const item = this.channels[channel][slot];

      // eslint-disable-next-line
      console.log(`toggling - channel: ${channel} slot: ${slot}`);

      item.selected = !item.selected;
      if (item.selected) {
        // eslint-disable-next-line
        console.log(`note: ${this.note} instrument: ${this.selectedWaveform}`);
        item.note = this.note;
        item.instrument = this.selectedWaveform;
        item.length = this.noteLength;
      } else {
        item.note = null;
        item.instrument = null;
        item.length = null;
      }
    },
    createChannel(channel, channelLength) {
      const note = {
        note: null,
        length: null,
        instrument: null,
        channel,
      };

      return [...Array(channelLength).keys()].map(index => ({
        ...note,
        index,
      }));
    },
    createSynth(type) {
      return new Tone.Synth({
        oscillator: { type },
      }).toMaster();
    },
    createSnareSynth() {
      return new Tone.NoiseSynth({
        noise: { type: 'white' },
        envelope: {
          attack: 0.005,
          decay: 0.1,
          sustain: 0,
        },
        volume: -8,
      }).toMaster();
    },
    createSynths() {
      const synths = waveforms.reduce((s, o) => {
        // eslint-disable-next-line
        s[o] = this.createSynth(o);
        return s;
      }, {});
      synths.snare = this.createSnareSynth();
      return synths;
    },
    createChannelSynths() {
      this.channelSynths = {
        channel0: this.createSynths(),
        channel1: this.createSynths(),
        channel2: this.createSynths(),
      };
    },
    play() {
      const slots = [...Array(this.channelLength).keys()];

      // eslint-disable-next-line
      this.loop = new Tone.Sequence(
        (time, slot) => {
          this.currentPosition =
            this.currentPosition === this.channelLength - 1
              ? 0
              : this.currentPosition + 1;

          // what is this and why
          const notes = [
            this.channels[0][slot],
            this.channels[1][slot],
            this.channels[2][slot],
          ];

          notes.forEach(note => {
            const synths = this.channelSynths[`channel${note.channel}`];

            switch (note.instrument) {
              case null:
                break;
              case 'snare':
                synths.snare.triggerAttackRelease();
                break;
              default:
                synths[note.instrument].triggerAttackRelease(
                  note.note,
                  note.length,
                );
                break;
            }
          });
        },
        slots,
        '4n',
      ).start();

      Tone.Transport.start();
    },
    stop() {
      Tone.Transport.stop();
      this.currentPosition = -1;
      if (this.loop !== null) {
        this.loop.dispose();
        this.loop = null;
      }
    },
  },
  computed: {
    songAsJson: {
      get() {
        return JSON.stringify(this.channels);
      },
      set(newSong) {
        this.channels = JSON.parse(newSong);
      },
    },
  },
  created() {
    this.createChannelSynths();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.tracker-container {
  display: grid;
  grid-template-columns: auto auto auto;
  width: 40%;
  margin: 0 auto;
  justify-content: center;
}
.np-container {
  display: grid;
  grid-template-columns: auto;
  grid-gap: 5px;
  padding: 10px;
  grid-auto-rows: 1fr;
  justify-content: center;
}
.track-container {
  display: grid;
  grid-template-columns: auto;
  grid-gap: 5px;
  padding-top: 10px;
  padding-bottom: 10px;
  padding-left: 2px;
  padding-right: 2px;
  grid-auto-rows: 1fr;
  justify-content: center;
}
.tracks-container {
  display: grid;
  grid-template-columns: auto auto auto;
  grid-auto-rows: 1fr;
  justify-content: center;
}
.grid-item {
  background-color: orange;
  border: 1px solid rgb(255, 255, 255);
  height: 20px;
  width: 50px;
}
.grid-item-selected {
  background-color: rgb(255, 155, 0);
}
.grid-item-unselected {
  background-color: rgba(255, 153, 0, 0.5);
}
.column-header {
  border: 1px solid rgb(255, 255, 255);
  height: 20px;
  width: 50px;
}
.column-header-active {
  background-color: green;
}
.column-header-inactive {
  background-color: black;
}
</style>
