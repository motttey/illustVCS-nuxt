<template>
  <div class="container">
    <div>
      <h1 class="title">
        illustVCS
      </h1>
      <h2 class="subtitle">
        version control system for drawing
      </h2>
      <canvas id="drawingCanvas" width="960" height="540"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Logo from '~/components/Logo.vue'

// import { Stage, Shape } from '@createjs/easeljs/dist/easeljs.cjs';
// import { Tween } from '@createjs/tweenjs/dist/tweenjs.cjs';

export default Vue.extend({
  components: {
    Logo
  },
  created: function(){
    if (process.client) {
      // import { Stage, Shape } from '@createjs/easeljs/dist/easeljs.cjs';
      // import { Tween } from '@createjs/tweenjs/dist/tweenjs.cjs';
      const easljs = require('@createjs/easeljs/dist/easeljs.cjs');
      const tweenjs = require('@createjs/tweenjs/dist/tweenjs.cjs');

      let stage = new easljs.Stage("drawingCanvas");
      let shape = new easljs.Shape();

      shape.graphics.beginStroke("DarkRed");

      shape.graphics
        .moveTo(100, 100)
        .lineTo(500, 500);

      stage.addChild(shape);
      stage.update();
    }

  }
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
