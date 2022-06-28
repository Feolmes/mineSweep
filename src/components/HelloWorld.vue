<template>
  <div class="mine-area" style="margin: 30px; display: inline-block;">
    <div class="time">⏲：{{ time }}</div>
    <div class="bug-count">bug剩余数： {{ HIGH_LEVEL - flagCnt }}</div>
    <div v-for="(item, index) in arrayToShow" :key="index" style="display: flex;">
      <div v-for="(subItem, subIndex) in item" :key="subIndex" @click.stop.prevent="openRect(index, subIndex)" @contextmenu.stop.prevent="markAsMine(index, subIndex)">
        <div v-if="arrayToRecordOperator[index][subIndex] === STATUS.OPEN" class="rect" >
          <img class="rect" :src="imageArray[subItem]" />
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.CLOSE">
          <div style="width: 28px; height: 28px; background: #aaaaaa; border: 1px solid;"></div>
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.FLAG" class="rect" >
          <img class="rect"  :src="flagImg" />
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.QUESTION" class="rect" >
          <img class="rect"  :src="questionImg" />
        </div>
        <div v-else-if="arrayToRecordOperator[index][subIndex] === STATUS.ERROR_FLAG" class="rect" >
          <img class="rect"  :src="errorFlagImg" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import numZeroImg from '@/assets/0.png';
import numOneImg from '@/assets/1.png';
import numTwoImg from '@/assets/2.png';
import numThreeImg from '@/assets/3.png';
import numFourImg from '@/assets/4.png';
import numFiveImg from '@/assets/5.png';
import numSixImg from '@/assets/6.png';
import numSevenImg from '@/assets/7.png';
import numEightImg from '@/assets/8.png';
import numBugImg from '@/assets/9.png';
import flagImg from '@/assets/flag.png';
import questionImg from '@/assets/notSureIsMine.png';
import errorFlagImg from '@/assets/error-flag.png';
// import Vue from 'vue';
const HIGH_LEVEL = 99;
const HIGH_LEVEL_RECT = [16, 30];
const MINE_NUM = 9;
const STATUS = {
  OPEN: 0,
  CLOSE: 1,
  FLAG: 2,
  QUESTION: 3,
  ERROR_FLAG: 4
};
export default {
  name: 'HelloWorld',
  data() {
    return {
      arrayToShow: [],
      arrayToRecordOperator: [],
      imageArray: [numZeroImg, numOneImg, numTwoImg, numThreeImg, numFourImg, numFiveImg, numSixImg, numSevenImg, numEightImg, numBugImg],
      STATUS,
      flagImg,
      questionImg,
      errorFlagImg,
      count: 0,
      isFirstRect: true,
      firstX: -1,
      firstY: -1,
      timer: null,
      time: 0,
      flagCnt: 0,
      HIGH_LEVEL
    }
  },
  created() {
    this.init();
  },
  methods: {
    init() {
      var array = new Array();
      var arrayOperator = new Array();
      this.count = 0;
      this.firstX = -1;
      this.firstY = -1;
      this.time = 0;
      this.flagCnt = 0;
      for (var i = 0; i < HIGH_LEVEL_RECT[0]; i++) {
        array[i] = [];
        arrayOperator[i] = [];
        for (var j = 0; j < HIGH_LEVEL_RECT[1]; j++) {
          array[i].push(0);
          arrayOperator[i].push(1);
        }
      }
      this.arrayToRecordOperator = arrayOperator;
      // 布雷
      for (var iterMine = 0; iterMine < HIGH_LEVEL; iterMine++) {
        var row = Math.floor(Math.random() * HIGH_LEVEL_RECT[0]);
        var col = Math.floor(Math.random() * HIGH_LEVEL_RECT[1]);
        while(array[row][col] !== 0) {
          row = Math.floor(Math.random() * HIGH_LEVEL_RECT[0]);
          col = Math.floor(Math.random() * HIGH_LEVEL_RECT[1]);
        }
        array[row][col] = 9;
      }
      // 计算数字
      for (var iterRow = 0; iterRow < HIGH_LEVEL_RECT[0]; iterRow ++) {
        for (var iterCol= 0; iterCol < HIGH_LEVEL_RECT[1]; iterCol++) {
          if (array[iterRow][iterCol] === MINE_NUM) continue;
          var left = iterRow - 1;
          var right = iterRow + 1;
          var up = iterCol - 1;
          var down = iterCol + 1;
          var roundArray = [[left, up], [left, iterCol], [left, down], [iterRow, up], [iterRow, down], [right, up], [right, iterCol], [right, down]];
          var count = 0;
          for (var iterRound = 0; iterRound < roundArray.length; iterRound++) {
            var iterRoundRow = roundArray[iterRound][0];
            var iterRoundCol = roundArray[iterRound][1];
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
      var array = new Array();
      for (var iterx = 0; iterx < HIGH_LEVEL_RECT[0]; iterx++) {
        array[iterx] = [];
        for (var itery = 0; itery < HIGH_LEVEL_RECT[1]; itery++) {
          array[iterx].push(0);
        }
      }
      var deltaX = this.firstX - index;
      var deltaY = this.firstY - subIndex;
      for (var iterx1 = 0; iterx1 < HIGH_LEVEL_RECT[0]; iterx1++) {
        for (var itery1 = 0; itery1 < HIGH_LEVEL_RECT[1]; itery1++) {
          if (this.arrayToShow[iterx1][itery1] === MINE_NUM) {
            var x = (iterx1 - deltaX + HIGH_LEVEL_RECT[0]) % HIGH_LEVEL_RECT[0];
            var y = (itery1 - deltaY + HIGH_LEVEL_RECT[1]) % HIGH_LEVEL_RECT[1];
            array[x][y] = MINE_NUM;
          }
        }
      }
       // 计算数字
      for (var iterRow = 0; iterRow < HIGH_LEVEL_RECT[0]; iterRow ++) {
        for (var iterCol= 0; iterCol < HIGH_LEVEL_RECT[1]; iterCol++) {
          if (array[iterRow][iterCol] === MINE_NUM) continue;
          var left = iterRow - 1;
          var right = iterRow + 1;
          var up = iterCol - 1;
          var down = iterCol + 1;
          var roundArray = [[left, up], [left, iterCol], [left, down], [iterRow, up], [iterRow, down], [right, up], [right, iterCol], [right, down]];
          var count = 0;
          for (var iterRound = 0; iterRound < roundArray.length; iterRound++) {
            var iterRoundRow = roundArray[iterRound][0];
            var iterRoundCol = roundArray[iterRound][1];
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
      if (this.arrayToRecordOperator[index][subIndex] === STATUS.CLOSE) {
        this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.FLAG)
        this.flagCnt++;
      } else if (this.arrayToRecordOperator[index][subIndex] === STATUS.FLAG) {
        this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.QUESTION)
        this.flagCnt--;
      } else if (this.arrayToRecordOperator[index][subIndex] === STATUS.QUESTION) {
        this.$set(this.arrayToRecordOperator[index], subIndex, STATUS.CLOSE)
      }
    },
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
  props: {
    msg: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
</style>
