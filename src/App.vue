<template>
  <Board
    :boardArray="boardArray"
    @clicked="handleClick"
    :mines="currentSize.mines - mineStock"
    :markedMines="marked"
    @marked="handleMark"
  />
  <button @click="restartGame">Restart</button>
</template>

<script>
import Board from "./components/Board.vue";

//  sizes: {
//         beginner: { height: 9, width: 9, mines: 10 },
//         intermediate: { height: 9, width: 16, mines: 40 },
//         advanced: { height: 16, width: 30, mines: 99 },
//       }

export default {
  name: "App",
  components: {
    Board,
  },
  data() {
    return {
      currentSize: { height: 16, width: 30, mines: 99 },
      boardArray: [], //2d array with square objects コマオブジェクトが入っている２次元配列
      squaresWithMines: [], //Used when placing numbers, at explosion 数字配置と地雷爆発の時に使う
      mineStock: null, //Used when placing mines 地雷を配置した時に使う
      nowPlaying: true,
      shownArray: [], //Save revealed squares in here 中身（数字とか）見せられたコマをここで保存
      marked: 0, //Number of flags placed つけられた旗の個数
    };
  },
  methods: {
    restartGame() {
      this.boardArray = [];
      this.squaresWithMines = [];
      this.mineStock = null;
      this.nowPlaying = true;
      this.shownArray = [];
      this.marked = 0;
      this.setUpBoard();
    },
    //left click 左クリック
    handleClick(row, column) {
      let square = this.boardArray[row][column];
      if (square.mine && !square.marked) {
        this.nowPlaying = false;
        this.squaresWithMines.forEach((mine) => (mine.revealed = true));
      }
      if (this.nowPlaying && !square.marked) {
        square.revealed = true;
        if (square.number === 0) {
          this.showSquares(row, column);
        }
      }
    },
    //right click：add flag 右クリック:旗をつける
    handleMark(row, column) {
      let square = this.boardArray[row][column];
      if (!square.revealed) {
        square.marked = !square.marked;
        square.marked ? this.marked++ : this.marked--;
      }
    },
    //Click reveals some squares クリックしてあるコマが中身を見せるやつ
    showSquares(row, column) {
      let square = this.boardArray[row][column];
      let containsZero = square.number === 0;
      let notInArray = this.shownArray.every(
        (square) => !(square.row === row && square.column === column)
      );
      let notAMine = !square.mine;
      //If zero and not in shownArray ValueがゼロでshownArrayに保存されていない場合
      if (containsZero && notInArray) {
        square.revealed = true;
        this.shownArray.push(square);
        //Do same thing to neighbors ゼロの場合は隣のコマも同様
        const neighbors = this.findNeighbors(row, column);
        neighbors.forEach((neighbor) =>
          this.showSquares(neighbor.row, neighbor.column)
        );
      }
      //If not zero, not mine, not in shownArray ゼロではないが地雷でもない、まだshownArrayに保存していない場合
      else if (notAMine && notInArray) {
        square.revealed = true;
        this.shownArray.push(square);
      } else return;
    },
    //Find neighbors, return in array  あるマスの隣にあるマスを見つけて、配列にいれて戻す
    findNeighbors(row, column) {
      let neighborsArray = [];
      for (let i = row - 1; i <= row + 1; i++) {
        for (let j = column - 1; j <= column + 1; j++) {
          //Prevent out of range errors Out of range  エラーにならないようにチェック, クリックされた本人も外す
          let rowOK = i >= 0 && i < this.currentSize.height;
          let columnOK = j >= 0 && j < this.currentSize.width;
          let isNotTheSquare = !(i == row && j == column);
          if (rowOK && columnOK && isNotTheSquare) {
            neighborsArray.push(this.boardArray[i][j]);
          }
        }
      }
      return neighborsArray;
    },
    //Not using up all the mines, but we usually are  確率でやっているので地雷が全て使われないことがある...
    getMine() {
      let { height, width, mines } = this.currentSize;
      let probability = mines / (height * width);
      let hasMine = !!probability && Math.random() <= probability;
      if (this.mineStock > 0 && hasMine) {
        this.mineStock--;
        return true;
      } else return false;
    },
    //After mines placed, set numbers 地雷を配置したあとコマに数字を追加
    addNumbers() {
      this.boardArray.forEach((row) =>
        row.forEach((square) => {
          if (square.mine) this.squaresWithMines.push(square);
        })
      );
      this.squaresWithMines.forEach((square) => {
        const neighbors = this.findNeighbors(square.row, square.column);
        neighbors.forEach((neighbor) => (neighbor.number += 1));
      });
    },
    setUpBoard() {
      this.mineStock = this.currentSize.mines;
      let { height, width } = this.currentSize;
      //Put square objects in 2d array  マスオブジェクトを２次元配列に入れる
      let bigArray = new Array(height);
      for (let i = 0; i < height; i++) {
        let newRow = [];
        for (let j = 0; j < width; j++) {
          let squareObj = {
            row: i,
            column: j,
            //If true comes back, is a mine  true戻ってきたら地雷である
            mine: this.getMine(),
            number: 0,
            revealed: false,
            marked: false,
          };
          newRow.push(squareObj);
        }
        bigArray[i] = newRow;
      }
      this.boardArray = bigArray;
      this.addNumbers();
    },
  },
  created() {
    this.setUpBoard();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
