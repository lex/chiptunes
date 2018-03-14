<template>
  <div>
    <h1>{{ title }}</h1>
    <div class="tracker-container">
      <div class="np-container">
        <div v-for="i in channelLength" :key="`header-${i - 1}`" :class="`column-header column-header-${i - 1 === currentPosition ? 'active' : 'inactive'}`">{{(i - 1).toString(16).toUpperCase()}}</div>
      </div>
      <div class="tracks-container">
        <div class="track-container" v-for="(channel, channelIndex) in channels" :key="channelIndex">
          <div v-for="(note, i) in channel" :class="`grid-item grid-item-${note.selected ? 'selected' : 'unselected'}`" :key="i" :id="`${note.channel}:${note.index}`" v-on:click="clicked">{{i}}</div>
        </div>
      </div>
      <div>
        <input type="radio" id="lead" value="lead" v-model="picked">
        <label for="lead">Lead</label>
        <br>
        <input type="radio" id="bass" value="bass" v-model="picked">
        <label for="bass">Bass</label>
        <br>
        <input type="radio" id="snare" value="snare" v-model="picked">
        <label for="snare">Snare</label>
        <br>
        <input v-model="note">
        <br>
        <input v-model="songAsJson">
      </div>
    </div>
  </div>
</template>

<script>
import Tone from 'tone';

export default {
  name: 'HelloWorld',
  data() {
    return {
      title: 'Chiptunes',
      channelLength: 30,
      currentPosition: -1,
      tileColor: 'orange',
      channels: [
        this.createChannel(0),
        this.createChannel(1),
        this.createChannel(2),
      ],
      picked: 'lead',
      note: 'C4',
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
        console.log(`note: ${this.note} instrument: ${this.picked}`);
        item.note = this.note;
        item.instrument = this.picked;
      } else {
        item.note = null;
        item.instrument = null;
      }
    },
    createChannel(channel) {
      // why doesn't this.channelLength work ???
      const channelLength = 30;

      const note = {
        note: null,
        length: null,
        selected: false,
        instrument: null,
        channel,
      };

      return [...Array(channelLength).keys()].map(index => ({
        ...note,
        index,
      }));
    },
    play() {
      const leadSynth = new Tone.Synth({
        oscillator: { type: 'triangle' },
      }).toMaster();

      const bassSynth = new Tone.Synth({
        oscillator: { type: 'triangle' },
      }).toMaster();

      const snareSynth = new Tone.NoiseSynth({
        noise: { type: 'white' },
        envelope: {
          attack: 0.005,
          decay: 0.1,
          sustain: 0,
        },
        volume: -8,
      }).toMaster();

      const slots = [...Array(this.channelLength).keys()];

      // eslint-disable-next-line
      const loop = new Tone.Sequence(
        (time, slot) => {
          this.currentPosition =
            this.currentPosition === this.channelLength - 1
              ? 0
              : this.currentPosition + 1;

          const notes = [
            this.channels[0][slot],
            this.channels[1][slot],
            this.channels[2][slot],
          ];

          notes.forEach(note => {
            switch (note.instrument) {
              case 'lead':
                leadSynth.triggerAttackRelease(note.note, '4n');
                break;
              case 'bass':
                bassSynth.triggerAttackRelease(note.note, '4n');
                break;
              case 'snare':
                snareSynth.triggerAttackRelease();
                break;
              default:
                break;
            }
          });
        },
        slots,
        '4n',
      ).start();

      Tone.Transport.start();
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
    this.play();
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
