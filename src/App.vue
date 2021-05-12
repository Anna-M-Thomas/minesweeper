<template>
  <Board
    :boardArray="boardArray"
    @clicked="handleClick"
    :mines="currentSize.mines - mineStock"
    :markedMines="marked"
    @marked="handleMark"
  />
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
      boardArray: [],
      squaresWithMines: [],
      mineStock: null,
      nowPlaying: true,
      shownArray: [],
      marked: 0,
    };
  },
  methods: {
    handleClick(row, column) {
      if (
        this.boardArray[row][column].mine &&
        !this.boardArray[row][column].marked
      ) {
        this.nowPlaying = false;
        this.squaresWithMines.forEach((mine) => (mine.revealed = true));
      }
      if (this.nowPlaying && !this.boardArray[row][column].marked) {
        this.boardArray[row][column].revealed = true;
        if (this.boardArray[row][column].number === 0) {
          this.showSquares(row, column);
        }
      }
    },
    handleMark(row, column) {
      this.boardArray[row][column].marked = !this.boardArray[row][column]
        .marked;
      this.boardArray[row][column].marked ? this.marked++ : this.marked--;
    },
    showSquares(row, column) {
      let containsZero = this.boardArray[row][column].number === 0;
      let notInArray = this.shownArray.every(
        (item) => !(item.row === row && item.column === column)
      );
      let notAMine = !this.boardArray[row][column].mine;
      //If you contain zero and you're not in shown array
      if (containsZero && notInArray) {
        //show your contents
        this.boardArray[row][column].revealed = true;
        //put in the array
        this.shownArray.push(this.boardArray[row][column]);
        //Do the same to your neighbors
        const neighbors = this.findNeighbors(row, column);
        neighbors.forEach((neighbor) =>
          this.showSquares(neighbor.row, neighbor.column)
        );
      }
      //if you're not zero, but you're not a mine and not in shown array
      else if (notAMine && notInArray) {
        //reveal and put in the array
        this.boardArray[row][column].revealed = true;
        this.shownArray.push(this.boardArray[row][column]);
      } else return;
    },
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
    //あるマスの隣にあるマスを見つけて、配列にいれて戻す
    findNeighbors(row, column) {
      let neighborsArray = [];
      for (let i = row - 1; i <= row + 1; i++) {
        for (let j = column - 1; j <= column + 1; j++) {
          //Out of rangeエラーにならないようにチェック, クリックされた本人（本コマ）も外す
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
    //Not using up all the mines, but we usually are...
    //確率でやっているので地雷が全て使われないことがある...
    getMine() {
      let { height, width, mines } = this.currentSize;
      let probability = mines / (height * width);
      let hasMine = !!probability && Math.random() <= probability;
      if (this.mineStock > 0 && hasMine) {
        this.mineStock--;
        return true;
      } else return false;
    },
  },
  created() {
    this.mineStock = this.currentSize.mines;
    let { height, width } = this.currentSize;
    //マスオブジェクトを２次元配列に入れる
    let bigArray = new Array(height);
    for (let i = 0; i < height; i++) {
      let newRow = [];
      for (let j = 0; j < width; j++) {
        let squareObj = {
          row: i,
          column: j,
          //true戻ってきたら地雷になる
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
