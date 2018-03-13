<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li>
        <a href="https://vuejs.org" target="_blank">
          Core Docs
        </a>
      </li>
      <li>
        <a href="https://forum.vuejs.org" target="_blank">
          Forum
        </a>
      </li>
      <li>
        <a href="https://chat.vuejs.org" target="_blank">
          Community Chat
        </a>
      </li>
      <li>
        <a href="https://twitter.com/vuejs" target="_blank">
          Twitter
        </a>
      </li>
      <br>
      <li>
        <a href="http://vuejs-templates.github.io/webpack/" target="_blank">
          Docs for This Template
        </a>
      </li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li>
        <a href="http://router.vuejs.org/" target="_blank">
          vue-router
        </a>
      </li>
      <li>
        <a href="http://vuex.vuejs.org/" target="_blank">
          vuex
        </a>
      </li>
      <li>
        <a href="http://vue-loader.vuejs.org/" target="_blank">
          vue-loader
        </a>
      </li>
      <li>
        <a href="https://github.com/vuejs/awesome-vue" target="_blank">
          awesome-vue
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
import Tone from 'tone';

export default {
  name: 'HelloWorld',
  data() {
    return {
      msg: 'Welcome to Your Vue.js App',
    };
  },
  methods: {
    playHelloWorldSong() {
      const synth = new Tone.Synth().toMaster();

      const notes = [
        { note: 'C4', length: '4n' },
        { note: 'C4', length: '4n' },
        { note: 'C4', length: '4n' },
        { note: 'E4', length: '4n' },
        { note: 'D4', length: '4n' },
        { note: 'D4', length: '4n' },
        { note: 'D4', length: '4n' },
        { note: 'F4', length: '4n' },
        { note: 'E4', length: '4n' },
        { note: 'E4', length: '4n' },
        { note: 'D4', length: '4n' },
        { note: 'D4', length: '4n' },
        { note: 'C4', length: '2n' },
        { note: null, length: null },
        { note: null, length: null },
      ];

      // eslint-disable-next-line
      const loop = new Tone.Sequence(
        (time, note) => {
          if (!note.note) return;
          synth.triggerAttackRelease(note.note, note.length);
        },
        notes,
        '4n',
      ).start();

      Tone.Transport.start();
    },
  },
  created() {
    this.playHelloWorldSong();
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
</style>
