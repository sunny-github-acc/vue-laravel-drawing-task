<!-- Please remove this file from your project -->
<template>
  <b-container>
    <b-row class="vh-100 text-center" align-h="center" align-v="center">
      <b-card
        v-if="loadCanvas"
        title="Draw"
        tag="title"
        style="max-width: 20rem"
        class="mb-2"
      >
        <b-card-text>Let's draw something nice! </b-card-text>

        <b-card class="mb-4">
          <v-stage
            ref="stage"
            :config="configKonva"
            @mousedown="handleMouseDown"
            @mousemove="handleMouseMove"
            @mouseup="handleMouseUp"
          >
            <v-layer ref="layer">
              <v-line
                v-for="(line, index) in lines"
                :key="index + 'line'"
                :config="{
                  stroke: 'black',
                  points: line.points,
                  strokeWidth: 2,
                }"
              />
              <v-rect
                v-for="(rec, index) in recs"
                :key="index + 'rec'"
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
                :key="index + 'circle'"
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
        </b-card>

        <div>
          <b-button
            :variant="isLine === true ? 'primary' : ''"
            v-on:click="changeTool('brush')"
            >Brush</b-button
          >
          <b-button
            :variant="isRec === true ? 'primary' : ''"
            v-on:click="changeTool('rec')"
            >Rectangle</b-button
          >
          <b-button
            :variant="isCircle === true ? 'primary' : ''"
            v-on:click="changeTool('circle')"
            >Circle</b-button
          >
        </div>

        <b-form-group
          id="fieldset-1"
          class="mt-4"
          description="Enter title to save your drawing"
          label-for="input-1"
          valid-feedback="Saved!"
          :invalid-feedback="invalidFeedback"
          :state="state"
        >
          <b-form-input
            id="input-1"
            v-model="title"
            :state="state"
            @keydown="onChange"
            trim
          ></b-form-input>
        </b-form-group>
        <b-button class="mt-2" variant="success" v-on:click="save()"
          >SAVE</b-button
        >
        <b-button
          id="show-btn"
          class="mt-2"
          variant="danger"
          @click="$bvModal.show('bv-modal')"
          >Reset</b-button
        >

        <b-modal id="bv-modal" hide-footer>
          <div class="d-block text-center">
            <h3>Are you sure you want to reset your drawing?</h3>
          </div>
          <b-button class="mt-3" variant="danger" block @click="reset($bvModal)"
            >Reset drawing</b-button
          >
        </b-modal>

        <b-form-group
          id="fieldset-2"
          class="mt-4"
          description="Load saved drawing by title"
          label-for="input-2"
          :invalid-feedback="invalidLoad"
          :state="stateLoad"
        >
          <b-form-input
            id="input-1"
            v-model="load"
            :state="stateLoad"
            @keydown="onChangeLoad"
            trim
          ></b-form-input>
        </b-form-group>
        <b-button class="mt-2" variant="success" v-on:click="loadDrawings()"
          >LOAD</b-button
        >
      </b-card>
    </b-row>
  </b-container>
</template>

<script>
import Vue from "vue";

export default {
  name: "Draw",
  computed: {
    state() {
      if (!this.showInputError) return null;
      if (this.title.length > 3 && this.title.length < 26) {
        return true;
      }
      return false;
    },
    invalidFeedback() {
      if (this.title.length < 5) {
        return "Enter at least 4 characters.";
      } else if (this.title.length > 25) {
        return "Enter no more than 25 characters.";
      }
    },
    stateLoad() {
      if (!this.showLoadError || this.loading) return null;
      if (this.load === this.download) {
        return true;
      }
      return false;
    },
    invalidLoad() {
      if (this.load != this.download) {
        return "Did not find such drawing";
      }
    },
  },
  data() {
    return {
      title: "",
      load: "",
      loading: false,
      download: "",
      showInputError: false,
      showLoadError: false,
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
      this.isDrawing = false;

      if (tool === "brush") {
        this.isLine = true;
      } else if (tool === "rec") {
        this.isRec = true;
      } else {
        this.isCircle = true;
      }
    },
    async save() {
      this.showInputError = true;
      const drawing = {
        lines: this.lines,
        recs: this.recs,
        circles: this.circles,
      };
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          title: this.title,
          drawing: JSON.stringify({ drawing }),
        }),
      };

      await fetch(`http://localhost:8000/api/drawings`, requestOptions).then(
        (response) => console.log(response)
      );
    },
    async loadDrawings() {
      this.showLoadError = true;
      this.loading = true;

      const requestOptions = {
        method: "GET",
        headers: { "Content-Type": "application/json" },
      };

      await fetch("http://localhost:8000/api/drawings", requestOptions)
        .then((response) => response.json())
        .then((data) => {
          data.map(({ title, drawing }) => {
            if (title === this.load) {
              const draw = JSON.parse(drawing).drawing;
              this.lines = draw.lines;
              this.recs = draw.recs;
              this.circles = draw.circles;
              this.download = title;
            }
          });
        })
        .catch((error) => console.error(error));

      this.loading = false;
    },
    onChange() {
      this.showInputError = false;
    },
    onChangeLoad() {
      this.showLoadError = false;
    },
    reset(bvModal) {
      bvModal.hide("bv-modal");
      this.lines = [];
      this.recs = [];
      this.circles = [];
    },
  },
};
</script>
