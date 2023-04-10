<!-- Please remove this file from your project -->
<template>
  <div
    class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0"
  >
    <div v-if="loadCanvas">
      <v-stage
        ref="stage"
        :config="configKonva"
        @mousedown="handleMouseDown"
        @mousemove="handleMouseMove"
        @mouseup="handleMouseUp"
      >
        <v-layer ref="layer">
          <v-line
            v-for="line in lines"
            :key="line.id"
            :config="{
              stroke: 'black',
              points: line.points,
              strokeWidth: 2,
            }"
          />
          <v-rect
            v-for="(rec, index) in recs"
            :key="index"
            :config="{
              x: Math.min(rec.startPointX, rec.startPointX + rec.width),
              y: Math.min(rec.startPointY, rec.startPointY + rec.height),
              width: Math.abs(rec.width),
              height: Math.abs(rec.height),
              fill: 'rgb(0,0,0,0)',
              stroke: 'black',
              strokeWidth: 2,
            }"
          />
          <v-circle
            v-for="(circle, index) in circles"
            :key="index"
            :config="{
              x: Math.min(
                circle.startPointX,
                circle.startPointX + circle.width
              ),
              y: Math.min(
                circle.startPointY,
                circle.startPointY + circle.height
              ),
              width: Math.abs(circle.width),
              height: Math.abs(circle.height),
              fill: 'rgb(0,0,0,0)',
              stroke: 'black',
              strokeWidth: 2,
            }"
          />
        </v-layer>
      </v-stage>
    </div>
    <button v-on:click="changeTool('brush')">Change tool to brush</button>
    <button v-on:click="changeTool('rec')">Change tool to rectangle</button>
    <button v-on:click="changeTool('circle')">Change tool to circle</button>
  </div>
</template>

<script>
import Vue from "vue";

export default {
  name: "Draw",
  data() {
    return {
      configKonva: {
        width: 200,
        height: 200,
      },
      lines: [],
      recs: [],
      circles: [],
      isLine: true,
      isRec: false,
      isCircle: false,
      isDrawing: false,
      loadCanvas: false,
    };
  },
  async mounted() {
    if (process.browser) {
      const VueKonva = await import("vue-konva");
      Vue.use(VueKonva);
      this.loadCanvas = true;
    }
  },
  methods: {
    handleMouseDown(event) {
      this.isDrawing = true;

      if (this.isLine) {
        const pos = event.target.getStage().getPointerPosition();
        this.lines = [...this.lines, { points: [pos.x, pos.y] }];
      } else {
        const pos = this.$refs.stage.getNode().getPointerPosition();
        const params = {
          startPointX: pos.x,
          startPointY: pos.y,
          width: 0,
          height: 0,
        };
        if (this.isRec) {
          this.setRecs([...this.recs, params]);
        } else {
          this.setCircles([...this.circles, params]);
        }
      }
    },
    handleMouseMove(event) {
      if (!this.isDrawing) {
        return;
      }

      if (this.isLine) {
        const stage = event.target.getStage();
        const point = stage.getPointerPosition();
        let lastLine = this.lines[this.lines.length - 1];
        lastLine.points = lastLine.points.concat([point.x, point.y]);
        this.lines.splice(this.lines.length - 1, 1, lastLine);
      } else {
        const point = this.$refs.stage.getNode().getPointerPosition();
        let current = this.isRec
          ? this.recs[this.recs.length - 1]
          : this.circles[this.circles.length - 1];
        current.width = point.x - current.startPointX;
        current.height = point.y - current.startPointY;
      }
    },
    handleMouseUp() {
      this.isDrawing = false;
    },
    setRecs(element) {
      this.recs = element;
    },
    setCircles(element) {
      this.circles = element;
    },
    changeTool(tool) {
      this.isLine = false;
      this.isRec = false;
      this.isCircle = false;

      if (tool === "brush") {
        this.isLine = true;
      } else if (tool === "rec") {
        this.isRec = true;
      } else {
        this.isCircle = true;
      }
    },
  },
};
</script>
