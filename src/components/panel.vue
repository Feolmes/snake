<template>
  <div class="hello">
    <div class="panel-area">
      <div v-for="(item, idx) in mapArr" :key="idx" class="item">
        <div v-for="(subItem, subIdx) in item" :key="subIdx">
          <div class="subItem" :class="{content: subItem===NO_COVERD, coverd: subItem===COVERD, food: subItem===FOOD, head: subItem===SNAKE_HEAD}"></div>
        </div>
      </div>
    </div>
    <div v-if="isOver" class="over-area">
      <div>游戏结束！！！</div>
      <button  @click.stop.prevent="restart">重新开始</button>
    </div>
  </div>
</template>

<script>
const WIDTH = 12;
const HEIGHT = 12;
const NO_COVERD = 1;
const COVERD = 2;
const FOOD = 3;
const SNAKE_HEAD = 4;
const DIRECTION = {
  UP: 0,
  DOWN: 2,
  LEFT: 1,
  RIGHT: 3
}
const KEY = {
  UP: 38,
  DOWN: 40,
  LEFT: 37,
  RIGHT: 39
}
export default {
  name: 'panel',
  created() {
    this.init();
    document.addEventListener('keydown', this.keyDown, false);
  },
  data() {
    return {
      WIDTH,
      HEIGHT,
      NO_COVERD,
      COVERD,
      FOOD,
      SNAKE_HEAD,
      mapArr: [],
      snakeArr: [],
      aheadInverval: null,
      speed: 500,
      direction: DIRECTION.UP,
      isOver: false
    }
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.keyDown, false);
  },
  methods: {
    restart() {
      this.init();
      this.isOver = false;
    },
    init() {
      this.initArr();
      this.initSnake();
      this.initFood();
      this.ahead();
    },
    initArr() {
      for (let i = 0; i < WIDTH; i++) {
        this.mapArr[i] = [];
        for (let j = 0; j < HEIGHT; j++) {
          this.mapArr[i].push(NO_COVERD);
        }
      }
    },
    initSnake() {
      let timeStamp = Number(new Date());
      let x = timeStamp % WIDTH, y = timeStamp % HEIGHT; 
      this.direction = timeStamp % 4;
      for (let i = 2; i >= 0; i--) {
        if (this.direction === DIRECTION.LEFT) {
          x++;
          if (x >= WIDTH) {
            x = 0;
          }
        } else if (this.direction === DIRECTION.RIGHT) {
          x--;
          if (x < 0) {
            x = WIDTH - 1;
          }
        } else if (this.direction === DIRECTION.UP) {
          y++;
          if (y >= HEIGHT) {
            y = 0;
          }
        } else {
          y--;
          if (y < 0) {
            y = HEIGHT - 1;
          }
        }
        this.snakeArr.push({x, y});
        this.drawSnake(COVERD)
      }
    },
    drawSnake(type = COVERD) {
      for (let i = 0; i < this.snakeArr.length; i++) {
        let item = this.snakeArr[i];
        if (type === COVERD) {
          if (i === 0) {
            this.mapArr[item.x][item.y] = SNAKE_HEAD;
          } else {
            this.mapArr[item.x][item.y] = COVERD;
          }
        } else {
          this.mapArr[item.x][item.y] = NO_COVERD;
        }
      }
    },
    initFood() {
      let timeStamp = Number(new Date());
      let foodDelta = timeStamp % (WIDTH * HEIGHT - this.snakeArr.length);
      let cnt = 0;
      for (let i = 0; i < WIDTH; i++) {
        for (let j = 0; j < HEIGHT; j++) {
          if (cnt === foodDelta && this.mapArr[i][j] === NO_COVERD) {
              this.mapArr[i][j] = FOOD;
              return;
          }
          if (this.mapArr[i][j] === NO_COVERD) {
            cnt++;
          }
        }
      }
    },
    ahead() {
      const that = this;
      that.aheadInverval = setInterval(() => {
        let first = that.snakeArr[0];
        let xDelta = this.direction === DIRECTION.RIGHT ? 1 : this.direction === DIRECTION.LEFT ? -1 : 0;
        let yDelta = this.direction === DIRECTION.DOWN ? 1 : this.direction === DIRECTION.UP ? -1 : 0;
        let newPushX = first.x + xDelta;
        if (newPushX >= WIDTH) {
          newPushX = 0;
        } else if (newPushX < 0) {
          newPushX = WIDTH - 1;
        }
        let newPushY = first.y + yDelta;
        if (newPushY >= HEIGHT) {
          newPushY = 0;
        } else if (newPushY < 0) {
          newPushY = HEIGHT - 1;
        }
        that.snakeArr.unshift({ x: newPushX, y: newPushY });
        if (this.mapArr[newPushX][newPushY] !== FOOD) {
          if (this.mapArr[newPushX][newPushY] === COVERD) {
            clearInterval(that.aheadInverval);
            that.aheadInverval = null;
            this.snakeArr = [];
            that.isOver = true;
          } else {
            let lastOne = that.snakeArr.pop();
            this.mapArr[lastOne.x][lastOne.y] = NO_COVERD;
          }
        } else {
          this.initFood();
        }
        that.mapArr[first.x][first.y] = COVERD;
        that.mapArr[newPushX][newPushY] = SNAKE_HEAD;
      }, that.speed);
    },
    keyDown(e) {
      if (this.aheadInverval && !this.isOver) {
        let btnDir = e.keyCode - KEY.LEFT;
        if ((btnDir - this.direction) % 2 !== 0) {
          this.direction = e.keyCode - KEY.LEFT;
        }
      }
    }
  }
}
</script>

<style scoped>
.hello {
  display: flex;
  flex-direction: column;
  height: 50vh;
}
.item {
  display: flex;
}
.subItem {
  border: 0.5px solid black;
  width: 30px;
  height: 30px;
}
.content {
  background-color: gray;
}
.coverd {
  background-color: red;
}
.food {
  /* background-color: yellow; */
  background-image: url('../assets/food.jpg');
  background-size: contain;
  background-position: center;

}
.head {
  background-color: purple;
}
.panel-area, .over-area {
  margin: auto;
}
.panel-area {
  flex: 8;
}
.over-area {
  flex: 1;
}
</style>
