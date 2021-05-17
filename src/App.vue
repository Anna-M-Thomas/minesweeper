<template>
  <TheHeader />
  <Board
    :boardArray="boardArray"
    @clicked="handleClick"
    :wonGame="wonGame"
    :mines="currentSize.mines"
    :markedMines="marked"
    @marked="handleMark"
  />
  <TheLevelPicker @changeSize="changeSize" />
  <button @click="restartGame">Restart</button>
</template>

<script>
import Board from "./components/Board.vue";
import TheHeader from "./components/TheHeader.vue";
import TheLevelPicker from "./components/TheLevelPicker";

export default {
  name: "App",
  components: {
    Board,
    TheHeader,
    TheLevelPicker,
  },
  data() {
    return {
      pickedSize: "intermediate",
      currentSize: null,
      boardArray: [], //2d array with square objects コマオブジェクトが入っている２次元配列
      squaresWithMines: [], //Squares with mines 地雷を持っているコマ
      nowPlaying: true,
      wonGame: false,
      shownArray: [], //Save revealed squares in here 中身（数字とか）見せられたコマをここで保存
      marked: 0, //Number of flags placed つけられた旗の個数
    };
  },
  watch: {
    //Check win every time a square is marked. If all mines are marked = win
    //This relies on user not marking every square
    //旗をつけるたびにwinをチェック。すべての地雷にマークが付いていればwinとします
    //旗を全コマにつけないことを前提
    marked() {
      let theyWin = this.squaresWithMines.every((square) => square.marked);
      if (theyWin) {
        this.wonGame = true;
        this.nowPlaying = false;
      }
    },
  },
  methods: {
    changeSize(size) {
      this.pickedSize = size;
      this.restartGame();
    },
    restartGame() {
      this.boardArray = [];
      this.squaresWithMines = [];
      this.nowPlaying = true;
      this.wonGame = false;
      this.shownArray = [];
      this.marked = 0;
      this.setUpBoard();
    },
    //left click 左クリック
    handleClick(row, column) {
      if (!this.nowPlaying) {
        return;
      }
      let square = this.boardArray[row][column];
      if (square.mine && !square.marked) {
        this.nowPlaying = false;
        this.squaresWithMines.forEach((mine) => (mine.revealed = true));
      }
      if (!square.marked) {
        square.revealed = true;
        if (square.number === 0) {
          this.showSquares(row, column);
        }
      }
    },
    //right click：add flag 右クリック:旗をつける
    handleMark(row, column) {
      if (!this.nowPlaying) {
        return;
      }
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
      let notMarked = !square.marked;
      //If zero and not in shownArray ValueがゼロでshownArrayに保存されていない場合
      if (containsZero && notInArray && notMarked) {
        square.revealed = true;
        this.shownArray.push(square);
        //Do same thing to neighbors ゼロの場合は隣のコマも同様
        const neighbors = this.findNeighbors(row, column);
        neighbors.forEach((neighbor) =>
          this.showSquares(neighbor.row, neighbor.column)
        );
      }
      //If not zero, not mine, not in shownArray ゼロではないが地雷でもない、まだshownArrayに保存していない場合
      else if (notAMine && notInArray && notMarked) {
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
    setSize() {
      switch (this.pickedSize) {
        case "beginner":
          this.currentSize = { height: 9, width: 9, mines: 10 };
          break;
        case "intermediate":
          this.currentSize = { height: 9, width: 16, mines: 40 };
          break;
        case "advanced":
          this.currentSize = { height: 16, width: 30, mines: 99 };
          break;
        default:
          break;
      }
    },
    //Add mines 地雷を配置
    addMines() {
      let mines = this.currentSize.mines;
      while (mines > 0) {
        let randomRow = Math.floor(Math.random() * this.currentSize.height);
        let randomColumn = Math.floor(Math.random() * this.currentSize.width);
        if (!this.boardArray[randomRow][randomColumn].mine) {
          this.boardArray[randomRow][randomColumn].mine = true;
          mines--;
        }
      }
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
      this.setSize();
      let { height, width } = this.currentSize;
      //Put square objects in 2d array  マスオブジェクトを２次元配列に入れる
      let bigArray = new Array(height);
      for (let i = 0; i < height; i++) {
        let newRow = [];
        for (let j = 0; j < width; j++) {
          let squareObj = {
            row: i,
            column: j,
            number: 0,
            revealed: false,
            marked: false,
          };
          newRow.push(squareObj);
        }
        bigArray[i] = newRow;
      }
      this.boardArray = bigArray;
      this.addMines();
      this.addNumbers();
    },
  },
  created() {
    document.title = "Minesweeper";
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
}

body {
  margin: 0;
}

button {
  padding: 0.5rem 1rem;
  margin: 0.5 rem;
}
</style>
