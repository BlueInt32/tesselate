<template>
  <div class="container" id="container">
    <div
      class="dot"
      v-for="dot in dots"
      :key="dot.id"
      :style="{
        top: 500 * dot.y + 'px',
        left: 500 * dot.x + 'px',
      }"
      :class="{ dragged: dot.dragged }"
    ></div>
    <div
      class="square"
      v-for="(i, index) in tab"
      :key="index"
      :class="{ highlight: isOn(index) }"
      :style="{
        width: squareSize + 'px',
        height: squareSize + 'px',
        lineHeight: squareSize + 'px',
      }"
    ></div>
  </div>
</template>

<script lang="js">
import Vue from 'vue';

export default Vue.extend({
    data() {
        return {
            n: 40,
            tab: [],
            dots: [
             { id: 'a', x: 0.3, y: 0.3, dragged: false },
             { id: 'b', x: 0.9, y: 0.8, dragged: false },
             { id: 'c', x: 0.2, y: 0.6, dragged: false },
            ],
            relX: 0,
            relY: 0,
            isClicking: false
        }
    },
    mounted() {
        // 100 -> 500px / sqrt(100);
        this.tab = [];
        for (let index = 0; index < this.n*this.n; index++) {
            this.tab.push(false);
        }
        let container = document.getElementById('container');
        container.addEventListener('mousedown', this.dotClicked);
        container.addEventListener('mouseup', this.dotReleased);
        container.addEventListener('mousemove', this.moving);
        this.relX = container.getBoundingClientRect().x;
        this.relY = container.getBoundingClientRect().y;
        this.tesselate();
    },
    computed: {
        squareSize: function() {
            return 500/this.n; 
        }
    },
    methods: {
        isOn:function(i) {
            return this.tab[i];
        },
        dotClicked: function (e) {
            this.isClicking = true;
            let actX = e.x - this.relX;
            let actY = e.y - this.relY;
            this.dots.forEach(dot => {
                if(Math.abs(actX - 500 * dot.x) < 25 && Math.abs(actY - 500 * dot.y) < 25) {
                    console.log('clicked', dot.id);
                    dot.dragged = true;
                }
            });

        },
        dotReleased: function () {
            this.isClicking = false;
            this.dots.forEach(dot => {
                if(dot.dragged) {
                    dot.dragged = false;
                }
            });

        },
        moving: function(e) {
            if(!this.isClicking) return;
            this.dots.forEach(dot => {
                if(dot.dragged) {
                    dot.x = (e.x - 25)/500;
                    dot.y = (e.y - 25)/500;
                }
            });
            this.tesselate();
        },
        tesselate: function() {
            var N = this.n*this.n;
            for (let i = 0; i < N; i++) {
                let pt = {
                    x: (i % this.n)/this.n,
                    y: i / N
                };
                this.tab[i] = this.pointInTriangle(
                    pt, 
                    this.dots[0], 
                    this.dots[1], 
                    this.dots[2]); 
            } 
        },
        sign: function(p1, p2, p3) {
            var adjust = 0.025;
            return (p1.x - p3.x - adjust) * (p2.y - p3.y) - (p2.x - p3.x) * (p1.y - p3.y - adjust);
        },
        pointInTriangle :function(pt, v1, v2, v3)
        {
            var d1, d2, d3, has_neg, has_pos;

            d1 = this.sign(pt, v1, v2);
            d2 = this.sign(pt, v2, v3);
            d3 = this.sign(pt, v3, v1);

            has_neg = (d1 < 0) || (d2 < 0) || (d3 < 0);
            has_pos = (d1 > 0) || (d2 > 0) || (d3 > 0);

            return !(has_neg && has_pos);
        }
    }
})
</script>

<style>
html {
  box-sizing: border-box;
  background: #000;
  font-family: Consolas;
  color: #444;
}

.container {
  width: 500px;
  height: 500px;
  border: 1px solid #444;
  position: relative;
}
.square {
  float: left;
  margin: 0;
  background: #111;
  /* border: px solid #000; */
  text-align: center;
}
.highlight {
  background: rgb(255, 0, 255);
  color: #000;
}
.dot {
  position: absolute;
  background: white;
  height: 20px;
  width: 20px;
  border-radius: 12px;
  border: 3px solid white;
}
.dragged {
  background: black;
  border: 3px solid white;
}
</style>
