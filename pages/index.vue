<template>
  <div class="container">
    <div>
      <h1 class="title">
        illustVCS
      </h1>
      <h2 class="subtitle">
        version control system for drawing
      </h2>
      <ul id="revisions">
        <li v-for="revision in all_revisions" :key="revision.key">
          {{ revision.revs }}
        </li>
      </ul>
      <canvas id="drawingCanvas" width="960" height="540"></canvas>
      <svg id="revisions_DAG" width="960" height="540"> </svg>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Logo from '~/components/Logo.vue'
// import Logo from '~/components/Logo.vue'

import * as d3_base from "d3";
// import * as d3dag from'd3-dag';

export default Vue.extend({
  components: {
    Logo
  },
  data: function() {
    return {
      stage: {},
      new_shape: {},
      shape: {},
      revisions: [], // DAGにする
      all_revisions: [],
      dag: {},
    }
  },
  methods: {
    handleDown(event) {
      const easljs = require('@createjs/easeljs/dist/easeljs.cjs');
      this.new_shape = new easljs.Shape();
      this.new_shape.name = this.new_shape.id.toString(); // findByIdがない...

      this.dag =  d3dag.dagConnect();

      this.new_shape.graphics
        .beginStroke("black");

      this.new_shape.graphics.moveTo(event.stageX, event.stageY) // 描画開始位置を指定
      this.stage.addChild(this.new_shape);

      if (process.client) {
        this.stage.addEventListener("stagemousemove", this.handleMove);
        this.stage.addEventListener("stagemouseup", this.handleUp);
      }
    },
    handleMove(event) {
      this.new_shape.graphics
        .lineTo(event.stageX, event.stageY);
    },
    handleUp(event) {
      this.new_shape.graphics
              .lineTo(event.stageX, event.stageY);

      let stroke = this.new_shape.graphics.endStroke();

      if (process.client) {
        this.stage.removeEventListener("stagemousemove", this.handleMove);
        this.stage.removeEventListener("stagemouseup", this.handleUp);
      }
      this.stage.update();
      this.revisions.push(this.new_shape.name);
    },
    handleUndo(event){
      if (this.revisions.length == 0) return;
      const name = this.revisions.pop();
      this.stage.removeChild(this.stage.getChildByName(name));
      this.stage.update();
    },
    saveRevision(event){
      let rev = this.stage.children.map(x => x.id);
      this.all_revisions.push(
        {
          key: rev.join(','),
          revs: rev
        }
      );

      // console.log(rev);
    },
    onTick() {
      this.stage.update(); // Stageの描画を更新
    }
  },
  created: function(){
    if (process.client) {
      // import { Stage, Shape } from '@createjs/easeljs/dist/easeljs.cjs';
      // import { Tween } from '@createjs/tweenjs/dist/tweenjs.cjs';
      const easljs = require('@createjs/easeljs/dist/easeljs.cjs');
      const tweenjs = require('@createjs/tweenjs/dist/tweenjs.cjs');

      this.stage = new easljs.Stage("drawingCanvas");
      // this.shape = new easljs.Shape();
      this.stage.addEventListener("stagemousedown", this.handleDown);
      document.addEventListener('keydown', (event) => {
        if (event.ctrlKey) {
          this.handleUndo(event);
        }
        else if (event.key === 's'){
          this.saveRevision(event);
        }
      }, false);
      // this.stage.update();

      easljs.Ticker.timingMode = easljs.Ticker.RAF;
      easljs.Ticker.addEventListener("tick", this.onTick);
    }
  },
})
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

</style>
