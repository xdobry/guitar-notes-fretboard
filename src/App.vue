<template>
  <div class="page-container">
    <md-app md-waterfall md-mode="fixed">
      <md-app-toolbar class="md-primary">
        <div class="md-title">Guitar Notes Fretboard App</div>
      </md-app-toolbar>
      <md-app-content>
        <div class="md-layout md-gutter">
        <div class="md-layout-item">
          <md-field>
            <label for="movie">Instrument</label>
            <md-select v-model="instrument" id="instrument" name="instrument" @md-selected="drawBoard">
                <md-option value="guitar">guitar</md-option>
                <md-option value="bass">bass</md-option>
                <md-option value="ukulele">ukulele</md-option>
            </md-select>
          </md-field>
        </div>
        <div class="md-layout-item">
          <md-field>
            <label for="movie">Scale</label>
            <md-select v-model="scale" id="scale" name="scale" @md-selected="drawBoard">
                <md-option value="major">major</md-option>
                <md-option value="minor">minor natural</md-option>
                <md-option value="minorHarmonic">minor harmonic</md-option>
                <md-option value="majorPentatonic">major pentatonic</md-option>
                <md-option value="minorPentatonic">minor pentatonic</md-option>
                <md-option value="all">all</md-option>
            </md-select>
          </md-field>
        </div>
        <div class="md-layout-item">
          <md-field>
            <label for="movie">Key</label>
            <md-select v-model="baseToneSymbol" id="baseTone" name="baseTone" @md-selected="drawBoard">
              <md-option value="0">c</md-option>
              <md-option value="1">c#</md-option>
              <md-option value="2">d</md-option>
              <md-option value="3">d#</md-option>
              <md-option value="4">e</md-option>
              <md-option value="5">f</md-option>
              <md-option value="6">f#</md-option>
              <md-option value="7">g</md-option>
              <md-option value="8">g#</md-option>
              <md-option value="9">a</md-option>
              <md-option value="10">a#</md-option>
              <md-option value="11">b</md-option>
            </md-select>
          </md-field>
        </div>
        <div class="md-layout-item">
          <md-checkbox v-model="onlyChordNotes" @change="drawBoard">Only Chord Notes</md-checkbox>
        </div>
        </div>
          <div class="md-layout-item">
            <div>
              <canvas id="canvas" width="1800px" height="400px"></canvas>
            </div>
            <div>
                Programmed by xdobry <a href="https://github.com/xdobry/guitar-notes-fredboard">source on github</a>. No cookies here!
            </div>
        </div>
      </md-app-content>
    </md-app>
  </div>
</template>

<script>

const notes = ['c','c#','d','d#','e','f','f#','g','g#','a','a#','b'];
const marksColors = ['red','green','blue'];

const instruments = {
  guitar: {
    startNote: 4,
    intervals: [0,5,10,15,19,24],
    fcount: 15,
  },
  bass: {
    startNote: 4,
    intervals: [0,5,10,15],
    fcount: 18,
  },
  ukulele: {
    startNote: 7,
    intervals: [0,5,9,14],
    fcount: 13,
  },
}

const scales = {
  major: {
    intervals: [0,2,4,5,7,9,11],
    marks: [0,4,7],
  },
  minor: {
    intervals: [0,2,3,5,7,8,10],
    marks: [0,3,7],
  },
  minorHarmonic: {
    intervals: [0,2,3,5,7,8,11],
    marks: [0,3,7],
  },
  majorPentatonic: {
    intervals: [0,2,4,7,9],
    marks: [0,4,7],
  },
  minorPentatonic: {
    intervals: [0,3,5,7,10],
    marks: [0,3,7],
  },
  all: {
    intervals: [0,1,2,3,4,5,6,7,8,9,10,11],
    marks: [0,7],
  },
}

function noteSymbol(index) {
  return notes[index%12];
}

export default {
  name: "App",
  data: () => {
    return {
      scale: 'major',
      baseToneSymbol: '7',
      instrument: 'guitar',
      onlyChordNotes: false,
    };
  },
  mounted() {
      var c = document.getElementById("canvas");
      var ctx = c.getContext("2d");    
      this.vueCanvas = ctx;
      this.drawBoard();
  },
  methods: {
    drawBoard() {
      this.vueCanvas.clearRect(0, 0, 1800, 400);
      // draw rect
      const intervals = instruments[this.instrument].intervals;
      const scount = intervals.length;
      const startNote = instruments[this.instrument].startNote;
      const sx = 80;
      const fw = 80;
      const fcount = instruments[this.instrument].fcount;
      const sh = 60;
      const fh = (scount-1)*sh;
      const sy = 30;
      let i;
      this.vueCanvas.strokeStyle = "brown";
      this.vueCanvas.lineWidth = 4;
      for (i = 0;i<fcount;i++) {
        this.vueCanvas.beginPath();
        this.vueCanvas.moveTo(sx+i*fw,sy-5);
        this.vueCanvas.lineTo(sx+i*fw,sy+fh+5);
        this.vueCanvas.stroke();  
      }
      this.vueCanvas.lineWidth = 2;
      this.vueCanvas.strokeStyle = "black";
      for (i = 0;i<scount;i++) {
        this.vueCanvas.beginPath();
        this.vueCanvas.moveTo(sx,sy+i*sh);
        this.vueCanvas.lineTo(sx+(fcount-1)*fw,sy+i*sh);
        this.vueCanvas.stroke();  
      }
      this.vueCanvas.font = "30px Verdana";
      const curScale = scales[this.scale];
      const baseTone = Number.parseInt(this.baseToneSymbol);
      for (let fi = 0;fi<fcount;fi++) {
        for (let si = 0;si<scount;si++) {
          const noteIndex = (startNote+fi+intervals[si])%12;
          const scaleIndex = curScale.intervals.indexOf((noteIndex+12-baseTone)%12);
          if (scaleIndex>=0) {
            const symbol = noteSymbol(startNote+fi+intervals[si]);
            const markIndex = curScale.marks.indexOf((noteIndex+12-baseTone)%12);
            if (markIndex>=0) {
              this.vueCanvas.beginPath();
              this.vueCanvas.fillStyle = marksColors[markIndex];
              this.vueCanvas.arc(sx-(fw/2)+fi*fw,sy+fh-si*sh,28,0,2*Math.PI);
              this.vueCanvas.fill();
              this.vueCanvas.fillStyle = 'black';
            }
            // console.log("notIndex: "+noteIndex+ " s: "+symbol+ " iof "+ scaleIndex);
            if (markIndex>=0 || !this.onlyChordNotes) {
              this.vueCanvas.fillText(symbol,sx-(fw/2)-(symbol.length-1)*8+fi*fw,sy+fh-si*sh+8);
            }
          }
        }
      }
      const fmarks = [3,5,7,9,12];
      for (let fmark of fmarks) {
        this.vueCanvas.fillText(fmark.toString(),sx-(fw/2)+fmark*fw,sy+fh+58);
      }
    }
  }
};
</script>

<style>
</style>
