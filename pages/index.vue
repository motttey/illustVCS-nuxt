<template>
  <div class="container">
    <div>
      <h1 class="title">
        illustVCS
      </h1>
      <h2 class="subtitle">
        version control system for drawing
      </h2>
      <div id="revisions">
        <ul>
          <li v-for="revision in all_revisions" :key="revision.key">
            {{ revision.revs }}
          </li>
        </ul>
      </div>
      <div id="layers">
        <span v-for="layer in all_stage_layers" :key="layer.name">
          レイヤー {{ all_stage_layers.indexOf(layer) }}
        </span>
      </div>
      <input id="inputColor" type="color" value="#000000">

      <div id="canvas_holder">
        <canvas id="layer1" width="960" height="540"></canvas>
        <canvas id="layer2" width="960" height="540"></canvas>
        <canvas id="drawingCanvas" width="960" height="540"></canvas>
      </div>

      <svg id="revisions_DAG" width="960" height="540"> </svg>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Logo from '~/components/Logo.vue'
import sha256 from 'crypto-js/sha256'

import * as d3_base from "d3";
// import * as d3dag from'd3-dag';


/*
想定するデータ形式
stage_layers = [
  layer1 {
    index = 0
    stage_layer = {}
    color = "#000000"
    undo_stack = []
    redo_stack = []
    redo_revs = {}
  },

]
*/

export default Vue.extend({
  components: {
    Logo
  },
  data: function() {
    return {
      layer_index: 0,
      stage: {},
      stage_layer: {},
      all_stage_layers: [],
      new_shape: {},
      layer1_shape: {},
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

      // this.dag =  d3dag.dagConnect();
      // redoを初期化
      this.redo_stack = []
      this.redo_revs = {}

      this.new_shape.graphics
        .beginStroke("black");

      this.new_shape.graphics.moveTo(event.stageX, event.stageY) // 描画開始位置を指定
      this.stage.addChild(this.new_shape);

      // 参照でコピーされるっぽい -> 格納時にレイヤー情報を付加する? (要検討)
      this.layer1_shape = this.new_shape;

      this.layer1_shape.graphics.beginStroke(this.setLayerColor());
      this.layer1_shape.name = this.new_shape.id.toString();

      this.stage_layer.addChild(this.layer1_shape); //

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
      this.stage_layer.update();
      this.all_stage_layers[this.layer_index].undo_stack.push(this.layer1_shape.name);
    },
    handleUndo(event){
      if (this.all_stage_layers[this.layer_index].undo_stack.length == 0) return;
      const name = this.all_stage_layers[this.layer_index].undo_stack.pop();
      this.all_stage_layers[this.layer_index].redo_stack.push(name);
      this.all_stage_layers[this.layer_index].redo_revs[name] = this.stage_layer.getChildByName(name);

      this.stage_layer.removeChild(this.stage_layer.getChildByName(name));
      this.stage_layer.update();
    },
    handleRedo(event){
      if (this.all_stage_layers[this.layer_index].redo_stack.length == 0) return;
      const name = this.all_stage_layers[this.layer_index].redo_stack.pop();
      this.all_stage_layers[this.layer_index].undo_stack.push(name);

      let found_rev = this.all_stage_layers[this.layer_index].redo_revs[name];
      if (found_rev.length == 0)
        console.log("revision not found")
      else
        this.stage_layer.addChild(found_rev);

      delete this.all_stage_layers[this.layer_index].redo_revs[name];
      this.stage_layer.update();
    },
    saveRevision(event){
      // TODO, レイヤ情報を履歴に追加
      let rev = this.all_stage_layers[this.layer_index].stage_layer.children.map(x => x.id);
      const hash = sha256(new Date().toString()).toString();
      console.log("new revision array: " + hash);
      this.all_revisions.push(
        {
          key: hash,
          revs: rev
        }
      );
      // console.log(rev);
    },
    changeLayerIndex(){
      this.layer_index = (this.layer_index == 0)? 1: 0;
    },
    setLayerColor(){
      this.all_stage_layers[this.layer_index].color = document.querySelector("#inputColor").value;
      return this.all_stage_layers[this.layer_index].color;
    },
    selectLayer() {
      console.log(this.all_stage_layers);
      return this.all_stage_layers[this.layer_index].stage_layer;
    },
    onTick() {
      this.stage.update(); // Stageの描画を更新
      this.stage_layer.update();
    }
  },
  created: function(){
    if (process.client) {
      // import { Stage, Shape } from '@createjs/easeljs/dist/easeljs.cjs';
      // import { Tween } from '@createjs/tweenjs/dist/tweenjs.cjs';
      const easljs = require('@createjs/easeljs/dist/easeljs.cjs');
      const tweenjs = require('@createjs/tweenjs/dist/tweenjs.cjs');

      this.stage = new easljs.Stage("drawingCanvas");

      // init
      const layer_size = 2;
      //
      for (let index = 0; index < layer_size; index++) {
        let layer = {}
        layer.index = 0;
        layer.stage_layer = new easljs.Stage("layer" + (index + 1).toString());
        layer.color = "#000000";
        layer.undo_stack = [];
        layer.redo_stack = [];
        layer.redo_revs = {};
        this.all_stage_layers.push(layer);
      }

      this.stage_layer = this.all_stage_layers[this.layer_index].stage_layer;

      // this.shape = new easljs.Shape();
      this.stage.addEventListener("stagemousedown", this.handleDown);
      document.addEventListener('keydown', (event) => {
        if (event.ctrlKey) {
          this.handleUndo(event);
        }
        else if (event.key === 'r'){
          this.handleRedo(event);
        }
        else if (event.key === 's'){
          this.saveRevision(event);
        }
        else if (event.key === 'l'){
          this.changeLayerIndex(event);
          this.stage_layer =  this.selectLayer();
        }
      }, false);

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

canvas {
  position: absolute;
  left: 0;
}
</style>
