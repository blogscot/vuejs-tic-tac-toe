<template>
  <div class="gameStatus" :class="gameStatusColor">
    {{ gameStatusMessage }}
    <table class="grid">
      <tr>
        <cell name="1"></cell>
        <cell name="2"></cell>
        <cell name="3"></cell>
      </tr>
      <tr>
        <cell name="4"></cell>
        <cell name="5"></cell>
        <cell name="6"></cell>
      </tr>
      <tr>
        <cell name="7"></cell>
        <cell name="8"></cell>
        <cell name="9"></cell>
      </tr>
    </table>
  </div>
</template>

<script>
import Cell from "./Cell";

const winConditions = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
  [1, 4, 7],
  [2, 5, 8],
  [3, 6, 9],
  [1, 5, 9],
  [3, 5, 7]
];

function initialState() {
  return {
    // can be O or X
    activePlayer: "O",
    // maintains the status of the game: turn or win or draw
    gameStatus: "turn",
    gameStatusMessage: `O's turn`,
    gameStatusColor: "statusTurn",
    // no. of moves played by both players in a single game (max = 9)
    moves: 0,
    cells: {
      1: "",
      2: "",
      3: "",
      4: "",
      5: "",
      6: "",
      7: "",
      8: "",
      9: ""
    }
  };
}

export default {
  components: {
    cell: Cell
  },
  created() {
    // listens for a strike made by the user on cell
    // it is called by the Cell component
    Event.$on("strike", cellNumber => {
      // sets either X or O in the clicked cell of the cell's array
      this.cells[cellNumber] = this.activePlayer;
      this.moves++;
      this.gameStatus = this.changeGameStatus();
      if (this.gameStatus != "win") {
        this.changePlayer();
        this.gameStatusMessage = `${this.activePlayer}'s turn`;
      }
    });

    // listens for a restart button press
    // the data of the component is reinitialized
    // it is called by the App component
    Event.$on("gridReset", () => {
      Object.assign(this.$data, initialState());
    });
  },
  methods: {
    // returns the game status to the gameStatus property
    changeGameStatus() {
      if (this.checkForWin()) {
        // trigger parent component to change the score
        Event.$emit("win", this.activePlayer);
        this.gameStatusColor = "statusWin";
        this.gameStatusMessage = `${this.activePlayer} Wins !`;
        Event.$emit("freeze");
        return "win";
      } else if (this.moves === 9) {
        this.gameStatusColor = "statusDraw";
        this.gameStatusMessage = "Draw !";
        return "draw";
      }
      return "turn";
    },
    changePlayer() {
      this.activePlayer = this.activePlayer === "O" ? "X" : "O";
    },
    // checks for possible win conditions from the data
    checkForWin() {
      for (let i = 0; i < winConditions.length; i++) {
        // gets a single condition wc from the whole array
        let wc = winConditions[i];
        let cells = this.cells;
        if (this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])) {
          return true;
        }
      }
      return false;
    },
    // helper function for comparing cell values
    areEqual() {
      var len = arguments.length;
      for (var i = 1; i < len; i++) {
        if (arguments[i] === "" || arguments[i] !== arguments[i - 1])
          return false;
      }
      return true;
    }
  },
  data() {
    return initialState();
  }
};
</script>

<style scoped>
.grid {
  background-color: #34495e;
  color: #fff;
  width: 100%;
  border-collapse: collapse;
}

.gameStatus {
  margin: 0px;
  padding: 15px;
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  background-color: #f1c40f;
  color: #fff;
  font-size: 1.4em;
  font-weight: bold;
}

.statusTurn {
  background-color: #f1c40f;
}

.statusWin {
  background-color: #2ecc71;
}

.statusDraw {
  background-color: #9b59b6;
}
</style>
