<template>
  <div class="mine-area">
    <div class="time">⏲：{{ time }}</div>
    <div class="bug-count">bug剩余数： {{ HIGH_LEVEL - flagCnt }}</div>
    <div v-for="(item, index) in arrayToShow" :key="index" style="display: flex;">
      <div v-for="(rectItemShowVal, subIndex) in item" :key="subIndex" @click.stop.prevent="openRect(index, subIndex)" @contextmenu.stop.prevent="markAsMine(index, subIndex)">
        <div v-if="arrayToRecordOperator[index][subIndex] === STATUS.OPEN" class="rect open-status">
          <div v-if="rectItemShowVal !== MINE_NUM" :style="`color: ${NUM_COLOR_ARR[rectItemShowVal]}`">
            {{ rectItemShowVal || ' ' }}
          </div>
          <div v-else class="bug-rect">
            {{ 'bug' }}
          </div>
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.CLOSE">
          <div class="close-rect"></div>
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.FLAG" class="rect">
          <div class="flag-rect">
            {{ '🚩' }}
          </div>
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.QUESTION" class="rect">
          <div class="question-rect">
            {{ '?' }}
          </div>
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.ERROR_FLAG" class="rect">
          <div class="error-flag-x">
            {{ '×' }}
          </div>
          <div class="error-flag-word">
            {{ 'bug' }}
          </div>
        </div>
        <!-- 把按压状态去掉先 -->
        <!-- <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.PRESS" class="rect">
          <div style="width: 28px; height: 28px; background: #999999; border: 1px solid;"></div>
        </div> -->
      </div>
    </div>
  </div>
</template>

<script>
const HIGH_LEVEL = 99;
const HIGH_LEVEL_RECT = [16, 30];
const MINE_NUM = 9;
const STATUS = {
  OPEN: 0,
  CLOSE: 1,
  FLAG: 2,
  QUESTION: 3,
  ERROR_FLAG: 4
  // PRESS: 5
};
const MOUSE_LEFT = 0;
const NUM_COLOR_ARR = ['gray', 'blue', 'forestgreen', 'crimson', 'cadetblue', 'saddlebrown', 'darkorange', 'darkgoldenrod', 'dark']
export default {
  name: 'HelloWorld',
  data() {
    return {
      arrayToShow: [],
      arrayToRecordOperator: [],
      STATUS,
      count: 0,
      isFirstRect: true,
      firstX: -1,
      firstY: -1,
      timer: null,
      time: 0,
      flagCnt: 0,
      HIGH_LEVEL,
      isLeftMouseDown: false,
      // mouseLocX: -1,
      // mouseLocY: -1,
      MINE_NUM,
      NUM_COLOR_ARR
    }
  },
  created() {
    this.init();
    const that = this;
    document.addEventListener('mousedown', function(e) {
      if (e.button === MOUSE_LEFT) {
        that.isLeftMouseDown = true;
      }
    });
    document.addEventListener('mouseup', function(e) {
      if (e.button === MOUSE_LEFT) {
        that.isLeftMouseDown = false;
      }
    });
  },
  methods: {
    init() {
      let array = new Array();
      let arrayOperator = new Array();
      this.count = 0;
      this.firstX = -1;
      this.firstY = -1;
      this.time = 0;
      this.flagCnt = 0;
      for (let i = 0; i < HIGH_LEVEL_RECT[0]; i++) {
        array[i] = [];
        arrayOperator[i] = [];
        for (let j = 0; j < HIGH_LEVEL_RECT[1]; j++) {
          array[i].push(0);
          arrayOperator[i].push(1);
        }
      }
      this.arrayToRecordOperator = arrayOperator;
      // 布雷
      for (let iterMine = 0; iterMine < HIGH_LEVEL; iterMine++) {
        let row = Math.floor(Math.random() * HIGH_LEVEL_RECT[0]);
        let col = Math.floor(Math.random() * HIGH_LEVEL_RECT[1]);
        while(array[row][col] === MINE_NUM) {
          row = Math.floor(Math.random() * HIGH_LEVEL_RECT[0]);
          col = Math.floor(Math.random() * HIGH_LEVEL_RECT[1]);
        }
        array[row][col] = MINE_NUM;
      }
      // 计算数字
      for (let iterRow = 0; iterRow < HIGH_LEVEL_RECT[0]; iterRow ++) {
        for (let iterCol= 0; iterCol < HIGH_LEVEL_RECT[1]; iterCol++) {
          if (array[iterRow][iterCol] === MINE_NUM) continue;
          let left = iterRow - 1;
          let right = iterRow + 1;
          let up = iterCol - 1;
          let down = iterCol + 1;
          let roundArray = [[left, up], [left, iterCol], [left, down], [iterRow, up], [iterRow, down], [right, up], [right, iterCol], [right, down]];
          let count = 0;
          for (let iterRound = 0; iterRound < roundArray.length; iterRound++) {
            let iterRoundRow = roundArray[iterRound][0];
            let iterRoundCol = roundArray[iterRound][1];
            if (iterRoundRow >= 0 && iterRoundRow < HIGH_LEVEL_RECT[0] && iterRoundCol >= 0 && iterRoundCol < HIGH_LEVEL_RECT[1]) {
              if (array[iterRoundRow][iterRoundCol] === MINE_NUM) {
                count++;
              }
            }
          }
          if (count === 0 && this.firstX === -1 && this.firstY === -1) {
            if (iterRow !== 0 && iterRow !== HIGH_LEVEL_RECT[0] -1 && iterCol !== 0 && iterCol !== HIGH_LEVEL_RECT[1] - 1) {
              this.firstX = iterRow;
              this.firstY = iterCol;
            }
          }
          array[iterRow][iterCol] = count;
        }
      }
      this.arrayToShow = array;
      this.isFirstRect = true;
    },

    // 打开方块的逻辑，还有bug
    openRect(index, subIndex) {
      if (index < 0 || index >= HIGH_LEVEL_RECT[0] || subIndex < 0 || subIndex >= HIGH_LEVEL_RECT[1]) return;
      if (this.arrayToRecordOperator[index][subIndex] !== STATUS.CLOSE) return;
      if (this.isFirstRect) {
        this.timer = setInterval(() => {
          this.time++;
        }, 1000)
        if (this.arrayToShow[index][subIndex] !== 0) {
          this.addjustRect(index, subIndex);
        }
        this.isFirstRect = false;
      }
      this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.OPEN)
      this.count++;
      if (this.count === HIGH_LEVEL_RECT[0] * HIGH_LEVEL_RECT[1] - HIGH_LEVEL) {
        setTimeout(() => {
          clearInterval(this.timer);
          localStorage.setItem('gamecnt', (Number(localStorage.getItem('gamecnt')) || 0) + 1)
          if (!localStorage.getItem('minTime') || this.time < localStorage.getItem('minTime')) {
            localStorage.setItem('minTime', this.time)
          }
          alert(`恭喜你赢了！时间： ${this.time}秒, 总局数：${localStorage.getItem('gamecnt')}, 记录：${localStorage.getItem('minTime')}`);
          this.init();
        }, 50)
        return;
      }
      if (this.arrayToShow[index][subIndex] === MINE_NUM) {
        localStorage.setItem('gamecnt', (Number(localStorage.getItem('gamecnt') || 0)) + 1)
        clearInterval(this.timer);
        this.showMineDistribute();
        setTimeout(() => {
          alert(`游戏失败！时间： ${this.time}秒, 总局数： ${localStorage.getItem('gamecnt')}, 记录：${localStorage.getItem('minTime')}`);
          this.init();
        }, 50)
        return;
      }
      if (this.arrayToShow[index][subIndex] === 0) {
        this.openRect(index, subIndex - 1);
        this.openRect(index, subIndex + 1);
        this.openRect(index + 1, subIndex - 1);
        this.openRect(index + 1, subIndex);
        this.openRect(index + 1, subIndex + 1);
        this.openRect(index - 1, subIndex - 1);
        this.openRect(index - 1, subIndex);
        this.openRect(index - 1, subIndex + 1);
      }
    },

    addjustRect(index, subIndex) {
      let array = new Array();
      for (let iterx = 0; iterx < HIGH_LEVEL_RECT[0]; iterx++) {
        array[iterx] = [];
        for (let itery = 0; itery < HIGH_LEVEL_RECT[1]; itery++) {
          array[iterx].push(0);
        }
      }
      let deltaX = this.firstX - index;
      let deltaY = this.firstY - subIndex;
      for (let iterx1 = 0; iterx1 < HIGH_LEVEL_RECT[0]; iterx1++) {
        for (let itery1 = 0; itery1 < HIGH_LEVEL_RECT[1]; itery1++) {
          if (this.arrayToShow[iterx1][itery1] === MINE_NUM) {
            let x = (iterx1 - deltaX + HIGH_LEVEL_RECT[0]) % HIGH_LEVEL_RECT[0];
            let y = (itery1 - deltaY + HIGH_LEVEL_RECT[1]) % HIGH_LEVEL_RECT[1];
            array[x][y] = MINE_NUM;
          }
        }
      }
       // 计算数字
      for (let iterRow = 0; iterRow < HIGH_LEVEL_RECT[0]; iterRow ++) {
        for (let iterCol= 0; iterCol < HIGH_LEVEL_RECT[1]; iterCol++) {
          if (array[iterRow][iterCol] === MINE_NUM) continue;
          let left = iterRow - 1;
          let right = iterRow + 1;
          let up = iterCol - 1;
          let down = iterCol + 1;
          let roundArray = [[left, up], [left, iterCol], [left, down], [iterRow, up], [iterRow, down], [right, up], [right, iterCol], [right, down]];
          let count = 0;
          for (let iterRound = 0; iterRound < roundArray.length; iterRound++) {
            let iterRoundRow = roundArray[iterRound][0];
            let iterRoundCol = roundArray[iterRound][1];
            if (iterRoundRow >= 0 && iterRoundRow < HIGH_LEVEL_RECT[0] && iterRoundCol >= 0 && iterRoundCol < HIGH_LEVEL_RECT[1]) {
              if (array[iterRoundRow][iterRoundCol] === MINE_NUM) {
                count++;
              }
            }
          }
          if (count === 0 && this.firstX === -1 && this.firstY === -1) {
            this.firstX = iterRow;
            this.firstY = iterCol;
          }
          array[iterRow][iterCol] = count;
        }
      }
      this.arrayToShow = array;
    },

    markAsMine(index, subIndex) {
      if (this.isLeftMouseDown) {
        if (this.arrayToRecordOperator[index][subIndex] !== STATUS.OPEN) return;
        let left = index - 1;
        let right = index + 1;
        let up = subIndex - 1;
        let down = subIndex + 1;
        let roundArray = [[left, up], [left, subIndex], [left, down], [index, up], [index, down], [right, up], [right, subIndex], [right, down]];
        let flagCnt = 0;
        for (let roundIterFlag = 0; roundIterFlag < roundArray.length; roundIterFlag++) {
          let roundIterX = roundArray[roundIterFlag][0];
          let roundIterY = roundArray[roundIterFlag][1];
          if (roundIterX >= 0 && roundIterX < HIGH_LEVEL_RECT[0] && roundIterY >= 0 && roundIterY < HIGH_LEVEL_RECT[1]) {
            if (this.arrayToRecordOperator[roundIterX][roundIterY] === STATUS.FLAG) {
              flagCnt++;
            }
          }
        }
        if (flagCnt !== this.arrayToShow[index][subIndex]) return;
        for (let roundIter = 0; roundIter < roundArray.length; roundIter++) {
          let x = roundArray[roundIter][0];
          let y = roundArray[roundIter][1];
          if (x >= 0 && x < HIGH_LEVEL_RECT[0] && y >= 0 && y < HIGH_LEVEL_RECT[1]) {
            this.openRect(x, y);
          }
        }
      } else {
        if (this.arrayToRecordOperator[index][subIndex] === STATUS.CLOSE) {
          this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.FLAG)
          this.flagCnt++;
        } else if (this.arrayToRecordOperator[index][subIndex] === STATUS.FLAG) {
          this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.QUESTION)
          this.flagCnt--;
        } else if (this.arrayToRecordOperator[index][subIndex] === STATUS.QUESTION) {
          this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.CLOSE)
        }
      }
    },

    // 失败后展示出所有bug
    showMineDistribute() {
      for (let iterX = 0; iterX < HIGH_LEVEL_RECT[0]; iterX++) {
        for (let iterY = 0; iterY < HIGH_LEVEL_RECT[1]; iterY++) {
          if (this.arrayToShow[iterX][iterY] === MINE_NUM && this.arrayToRecordOperator[iterX][iterY] !== STATUS.FLAG) {
            this.$set(this.arrayToRecordOperator[iterX], iterY, STATUS.OPEN);
          } else if (this.arrayToShow[iterX][iterY] !== MINE_NUM && this.arrayToRecordOperator[iterX][iterY] === STATUS.FLAG) {
            this.$set(this.arrayToRecordOperator[iterX], iterY, STATUS.ERROR_FLAG);
          }
        }
      }
    }
  },
  destroyed() {
    document.removeEventListener('mousedown');
    document.removeEventListener('mouseup');
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.mine-area {
  margin: 30px; 
  display: inline-block;
}
.time {
  margin-bottom: 10px;
}
.bug-count {
  margin-bottom: 10px;
}
.rect {
  width: 30px; 
  height: 30px;
  user-select: none;
  -webkit-user-drag: none;
}
.open-status {
  font-size: 22px; 
  background: #dddddd; 
  line-height: 30px; 
  font-weight: 900;
}
.bug-rect {
  font-size: 15px; 
  color: red;
}
.close-rect {
  width: 28px; 
  height: 28px; 
  background: #aaaaaa; 
  border: 1px solid;
}
.flag-rect {
  background: #aaaaaa; 
  line-height: 30px; 
  font-weight: 900;
}
.question-rect {
  font-size: 28px; 
  color: black; 
  background: #aaaaaa; 
  line-height: 30px; 
  font-weight: 900;
}
.error-flag-x {
  position: absolute; 
  font-size: 35px; 
  color: red; 
  background: transparent; 
  width: 30px; 
  line-height: 30px; 
  transform: scale(2, 1);
}
.error-flag-word {
   font-size: 15px; 
   color: black; 
   background: #dddddd; 
   line-height: 30px; 
   font-weight: 900;

}
</style>
