<template>
  <div class="container">
    <h1>Boggle Game</h1>
    <div>
      <label for="gridSize">Select Grid Size:</label>
      <select v-model="gridSize" @change="initializeBoard">
        <option value="4">4x4</option>
        <option value="5">5x5</option>
      </select>
    </div>
    <div class="word-container">
      <div
        v-for="(row, rowIndex) in board"
        :key="rowIndex"
        class="grid-container grid-row"
        :style="{ gridTemplateColumns: `repeat(${gridSize}, 1fr)` }"
      >
        <div
          v-for="(letterObj, colIndex) in row"
          :key="colIndex"
          class="cell"
          :class="{
            selected: isSelected(rowIndex, colIndex),
            correct: letterObj.correct,
            'orange-background': letterObj.orangeBackground,
          }"
          @click="selectCell(rowIndex, colIndex)"
        >
          {{ letterObj.letter }}
        </div>
      </div>
    </div>
    <button @click="submitWord">Submit Word</button>
    <div v-if="submittedWords.length > 0">
      <h2>Submitted Words:</h2>
      <ul>
        <li v-for="(word, index) in submittedWords" :key="index">{{ word }}</li>
      </ul>
    </div>
    <div v-if="selectedWord.length > 0">
      <h2>Selected Word:</h2>
      <p>{{ selectedWord }}</p>
    </div>
  </div>
</template>

<script>
import { dictionary } from "@/Datas/wordsData";

export default {
  data() {
    return {
      gridSize: 4,
      board: [],
      submittedWords: [],
      minLength: 3,
      maxLength: 4,
      dictionary: dictionary,
      selectedWord: "",
      selectedCells: [],
    };
  },
  methods: {
    initializeBoard() {
      const alphabet = "abcdefghijklmnopqrstuvwxyz";
      let letters = [];

      for (let i = 0; i < this.gridSize * this.gridSize; i++) {
        const randomIndex = Math.floor(Math.random() * alphabet.length);
        letters.push(alphabet[randomIndex]);
      }

      const filteredWords = this.filterWordsByLength(
        this.dictionary,
        this.minLength,
        this.maxLength
      );

      for (let i = filteredWords.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [filteredWords[i], filteredWords[j]] = [filteredWords[j], filteredWords[i]];
      }

      const wordsToPlace = filteredWords.slice(0, 3);
      console.log(wordsToPlace);
      wordsToPlace.forEach((word) => {
        const direction = Math.random() > 0.5 ? "horizontal" : "vertical";
        const position = this.getRandomPosition(this.gridSize, word.length, direction);
        this.placeWordOnBoard(letters, word, position, direction);
      });

      this.board = this.wordArray(letters, this.gridSize).map((row) =>
        row.map((letter) => ({ letter, correct: false, orangeBackground: false }))
      );
    },

    getRandomPosition(gridSize, wordLength, direction) {
      const maxPosition = gridSize - (direction === "horizontal" ? wordLength : 1);

      return {
        row: Math.floor(Math.random() * gridSize),
        col: Math.floor(Math.random() * maxPosition),
        direction: direction,
      };
    },

    placeWordOnBoard(letters, word, position, direction) {
      const { row, col } = position;
      const wordLength = word.length;

      if (direction === "horizontal") {
        const startingCol = Math.floor(Math.random() * (this.gridSize - wordLength + 1));
        for (let i = 0; i < wordLength; i++) {
          letters[row * this.gridSize + startingCol + i] = word[i];
        }
      } else if (direction === "vertical") {
        const startingRow = Math.floor(Math.random() * (this.gridSize - wordLength + 1));
        for (let i = 0; i < wordLength; i++) {
          letters[(startingRow + i) * this.gridSize + col] = word[i];
        }
      }
    },

    wordArray(array, size) {
      const result = [];
      for (let i = 0; i < array.length; i += size) {
        result.push(array.slice(i, i + size));
      }
      return result;
    },

    submitWord() {
      if (this.selectedWord.length > 0 && this.isValidWord()) {
        const wordToSubmit = this.selectedWord.toLowerCase();

        if (this.dictionary.includes(wordToSubmit)) {
          this.submittedWords.push(wordToSubmit);
          this.markSelectedCellsAsCorrect();
        } else {
          this.markSelectedCellsAsIncorrect();
        }

        this.clearSelectedWord();
      } else {
        this.clearSelectedWord();
      }
    },

    isValidWord() {
      if (this.selectedCells.length < 2) {
        return false;
      }

      const firstCell = this.selectedCells[0];
      const secondCell = this.selectedCells[1];

      if (firstCell.row === secondCell.row || firstCell.col === secondCell.col) {
        return true;
      }
      alert("Please,Select in valid formet");
      return false;
    },

    markSelectedCellsAsCorrect() {
      this.selectedCells.forEach(({ row, col }) => {
        this.board[row][col].correct = true;
        this.board[row][col].orangeBackground = true;
      });
    },

    markSelectedCellsAsIncorrect() {
      this.selectedCells.forEach(({ row, col }) => {
        this.board[row][col].correct = false;
        this.board[row][col].orangeBackground = false;
      });
    },

    filterWordsByLength(wordsArray, minLength, maxLength) {
      return wordsArray.filter(
        (word) => word.length >= minLength && word.length <= maxLength
      );
    },

    selectCell(row, col) {
      const cellIndex = this.selectedCells.findIndex(
        (cell) => cell.row === row && cell.col === col
      );

      if (cellIndex !== -1) {
        this.selectedCells.splice(cellIndex, 1);
        if (Array.isArray(this.selectedWord)) {
          this.selectedWord.splice(cellIndex, 1);
        } else {
          this.selectedWord = this.selectedWord.split("");
          this.selectedWord.splice(cellIndex, 1).join(" ");
        }
      } else {
        this.selectedCells.push({ row, col });
        this.selectedWord += this.board[row][col].letter;
      }
    },

    clearSelectedWord() {
      this.selectedWord = "";
      this.selectedCells = [];
    },

    isSelected(row, col) {
      return this.selectedCells.some((cell) => cell.row === row && cell.col === col);
    },
  },
  mounted() {
    this.initializeBoard();
  },
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.grid-container {
  display: grid;
  gap: 5px;
  margin-top: 5px;
}

.grid-row {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(40px, 1fr));
  gap: 5px;
}

.cell {
  border: 1px solid #ccc;
  height: 40px;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  background-color: #f0f0f0;
  transition: background-color 0.3s ease;
  cursor: pointer;
}

.cell:hover {
  background-color: #e0e0e0;
}

.cell.selected {
  background-color: #4caf50;
  color: white;
}

.cell.correct.orange-background {
  background-color: orange;
  color: white;
}

label {
  margin-right: 10px;
}

select {
  padding: 5px;
  font-size: 14px;
}

button {
  margin-top: 10px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #45a049;
}

h1,
h2 {
  text-align: center;
}

h2 {
  margin-top: 20px;
  font-size: 18px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
  font-size: 14px;
}

.word-container {
  width: 25vw;
}
</style>
