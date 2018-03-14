<template>
  <div>
    <h1>{{ title }}</h1>
    <div class="grid-container">
      <div v-for="i in channelLength" :key="`header-${i - 1}`" :class="`column-header-${i - 1 === currentPosition ? 'active' : 'inactive'}`">{{i - 1}}</div>
      <div v-for="(note, i) in channels.reduce((a, b) => a.concat(b), [])" :class="`grid-item grid-item-${note.selected ? 'selected' : 'unselected'}`" :key="i" :id="i" v-on:click="clicked">{{i}}</div>
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
      channelLength: 16,
      currentPosition: -1,
      tileColor: 'orange',
      channels: [
        this.createChannel(),
        this.createChannel(),
        this.createChannel(),
      ],
    };
  },
  methods: {
    clicked(e) {
      const id = e.target.id;
      const channel = Math.floor(id / this.channelLength);
      const slot = Math.floor(id - channel * this.channelLength);
      const item = this.channels[channel][slot];

      item.selected = !item.selected;
      item.note = item.selected ? 'C4' : null;
    },
    createChannel() {
      // why doesn't this.channelLength work ???
      const channelLength = 16;
      const note = { note: null, length: null, selected: false };
      return [...Array(channelLength).keys()].map(() => ({ ...note }));
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

          const leadNote = this.channels[0][slot];
          const bassNote = this.channels[1][slot];
          const snareNote = this.channels[2][slot];

          if (leadNote.note) {
            leadSynth.triggerAttackRelease('C4', '4n');
          }

          if (bassNote.note) {
            bassSynth.triggerAttackRelease('C2', '4n');
          }

          if (snareNote.note) {
            snareSynth.triggerAttackRelease();
          }
        },
        slots,
        '4n',
      ).start();

      Tone.Transport.start();
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
.grid-container {
  display: grid;
  grid-template-columns: auto auto auto auto auto auto auto auto auto auto auto auto auto auto auto auto;
  grid-gap: 10px;
  padding: 10px;
  grid-auto-rows: 1fr;
  width: 80%;
  margin: 0 auto;
  justify-content: center;
}
.grid-item {
  background-color: orange;
  border: 1px solid black;
  height: 50px;
  width: 50px;
}
.grid-item-selected {
  background-color: rgb(255, 155, 0);
}
.grid-item-unselected {
  background-color: rgba(255, 153, 0, 0.5);
}
.column-header-active {
  background-color: green;
}
.column-header-inactive {
  background-color: black;
}
</style>
