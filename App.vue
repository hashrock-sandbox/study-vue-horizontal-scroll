<template>
  <svg
    tabindex="0"
    :width="w"
    :height="h"
    @wheel="wheel"
    @pointerdown="startDrag"
    @pointermove="onDrag"
    @pointerup="stopDrag"
  >
    <polyline :points="points" />
    <line v-for="(tick, index) in ticks" :key="index" :x1="tick" :y1="0" :x2="tick" :y2="h" />

    <rect
      @pointerdown.stop="onStartDragScroll"
      @pointermove.stop="onDragScroll"
      @pointerup.stop="stopDrag"
      :x="scroll.x"
      :y="h-12"
      height="10"
      :width="scroll.w"
      rx="5"
      fill="rgba(0,0,0, 0.5)"
    />
  </svg>
</template>

<script lang="ts">
import Vue from "vue";
import * as scale from "d3-scale";

function generatePolyline(data, scale, height) {
  return (
    `${scale(0)},${height} ` +
    data
      .map((item: number, i: number) => {
        return `${scale(i)},${item}`;
      })
      .join(" ") +
    ` ${scale(data.length)}, ${height}`
  );
}

function generateRandomData(h) {
  const result = [];
  for (let i = 0; i < 300; i++) {
    result.push(
      ((Math.random() + Math.random() + Math.random() + Math.random()) / 4) * h
    );
  }
  return result;
}

export default Vue.extend({
  data() {
    return {
      w: 400,
      h: 200,
      dat: <number[]>[],
      cx: 100,
      zoom: 2,
      drag: null,
      timerange: [0, 400]
    };
  },
  computed: {
    scrollScale() {
      return scale
        .scaleLinear()
        .domain(this.timerange)
        .range([0, this.w]);
    },
    scroll() {
      return {
        x: this.scrollScale(this.x1),
        w: this.scrollScale(this.x2 - this.x1)
      };
    },
    x1(): number {
      return this.cx - this.w / 2 / this.zoom;
    },
    x2(): number {
      return this.cx + this.w / 2 / this.zoom;
    },
    ticks() {
      return this.scale.ticks(5).map(this.scale);
    },
    points(): string {
      return generatePolyline(this.dat, this.scale, this.h);
    },
    scale() {
      return scale
        .scaleLinear()
        .domain([this.x1, this.x2])
        .range([0, this.w]);
    }
  },
  methods: {
    wheel(event: MouseWheelEvent) {
      this.zoom =
        this.zoom + (-event.deltaY * (event.deltaMode ? 120 : 1)) / 500;
      if (this.zoom <= 0) {
        this.zoom = 0.05;
      }
    },
    startDrag(e: PointerEvent) {
      this.$el.setPointerCapture(e.pointerId);
      this.drag = { x: e.offsetX };
    },
    onStartDragScroll(e: PointerEvent) {
      (e.currentTarget as HTMLElement).setPointerCapture(e.pointerId);
      this.drag = { x: e.offsetX };
    },
    onDragScroll(e: PointerEvent) {
      if (this.drag) {
        const move = e.offsetX - this.drag.x;
        this.drag = { x: e.offsetX };
        this.cx = this.cx + this.scrollScale.invert(move);
      }
    },
    onDrag(e: PointerEvent) {
      if (this.drag) {
        //前回のpointer eventのxからの差分だけを出していく
        //この時点ではpx単位で起きたことだけを考えればよい
        const move = e.offsetX - this.drag.x;
        this.drag = { x: e.offsetX };

        //こっちは時間単位の世界
        // +方向にマウスが動いたら、基準点は-方向に移動する
        this.cx = this.cx - move / this.zoom;
      }
    },
    stopDrag(e: PointerEvent) {
      this.drag = null;
    }
  },
  mounted() {
    //ランダムデータ
    this.dat = generateRandomData(this.h);
  }
});
</script>
<style scoped>
svg {
  background: white;
  cursor: normal;
}
polyline {
  fill: #e7ece7;
  stroke: #793;
}
line {
  stroke: #d5ded5;
}
</style>