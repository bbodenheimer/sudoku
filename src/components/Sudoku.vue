/* eslint-disable vue/no-parsing-error */
<template>
  <div class="sudoku">
    <div class="row">
    <h2>Sudoku</h2>

    <!-- from computed -->
    <strong>{{ formattedTime }} </strong>

      <!-- display different difficulties in dropdown and change puzzle -->
      <select v-model="difficulty" @change="generatePuzzle()">
        <option
          v-for="(display, level) in levels" :key="level"
          :value="level"
          >
          {{ display }}
        </option>
      </select>
    </div>

    <div class="grid">
      <!-- to iterate over the rows and cells-->
      <div
        class="row"
        v-for="(row, rowIndex) in puzzle" :key="rowIndex"
        >
        <!-- border-right/bottom will only appear if what is to the right of the : is true, adds 3x3 grid lines -->
        <!-- original cells (not filled in by user) will be bolded -->
        <!-- when cell is clicked it will become active -->
        <!-- invalid is invalid entry, is only checked if contains value -->
        <div
          class="cell" :class="{
            'border-right': colIndex === 2 || colIndex === 5,
            'border-bottom': rowIndex === 2 || rowIndex === 5,
            'original': cell.original,
            'active': activeRow === rowIndex && activeCol === colIndex,
            'invalid': cell.value && isCellInvalid(rowIndex, colIndex, cell.value)
            }"
          v-for="(cell, colIndex) in row" :key="colIndex"
          @click="setCellActive(rowIndex, colIndex, cell.original)"
          >
          <!-- display value of each cell -->
          {{ cell.value }}
        </div>
      </div>
    </div>

    <!-- buttons for input, have to add one bc zero-based array -->
    <!-- buttons not clickable when no active valid cell -->
    <div class="row">
      <button
        type="button"
        class="btn"
        v-for="value in Array(9).keys()" :key="value"
        :disabled="activeRow === -1 || activeCol === -1"
        @click="setCellValue(value + 1)"
        >
        {{ value + 1 }}
      </button>
    </div>
  </div>
</template>

<script>
// import sudoku the OBJECT from the parent object in directory
import { sudoku } from '../JS/sudoku'

export default {
  name: 'Sudoku',
  data () {
    return {
      // where puzzle is stored
      puzzle: [],
      difficulty: 'easy',

      // to track active cell
      activeRow: -1,
      activeCol: -1,

      // getting different levels from sudoku.js
      // bc 'very-hard' contains '-' must have 's
      levels: {
        'easy': 'Easy',
        'medium': 'Medium',
        'hard': 'Hard',
        'very-hard': 'Very Hard',
        'insane': 'Insane',
        'inhuman': 'Inhuman'
      },
      seconds: 0,
      timer: null
    }
  },
  computed: {
    // formatting the timer
    formattedTime () {
      // get mins from total secs / 60
      let min = Math.floor(this.seconds / 60)
      // get secs from total secs % 60
      let sec = this.seconds % 60

      // always display as 00:00
      if (min < 10) {
        min = `0${min}`
      }
      if (sec < 10) {
        sec = `0${sec}`
      }

      // return timer
      return `${min}:${sec}`
    }
  },
  methods: {
    // get puzzle from imported sudoku library
    generatePuzzle() {
      const boardString = sudoku.generate(this.difficulty)
      // turns board into 9x9 array grid, one array of 9 elements, each element is another array of 9 characters
      this.puzzle = sudoku.board_string_to_grid(boardString)
        // turn into an array of arrays of objects from array of array of strings
        .map(row => {
          return row.map(cell => {
            return {
              // if cell is not a '.' turn into int
              value: cell !== '.' ? parseInt(cell) : null,
              // if cell is a number not '.' it is part of the original puzzle
              original: cell !== '.'
            }
          })
        })
        // reset timer on generated puzzle and increment every second
        this.seconds = 0
        clearInterval(this.timer)
        this.timer = setInterval(() => {
          this.seconds += 1
        }, 1000)
    },
    // select active cell
    setCellActive (row, col, original) {
      if (original) {
        return
      }

      // deselect cell if selected
      if (this.activeRow === row && this.activeCol === col) {
        this.activeRow = -1
        this.activeCol = -1
        return
      }

      // if not an original cell and not currently active, reassign
      this.activeRow = row
      this.activeCol = col
    },

    // user input to cell, activeRow/Col give x/y position, reset selected cell
    setCellValue (value) {
      this.puzzle[this.activeRow][this.activeCol].value = value
      this.activeRow = -1,
      this.activeCol = -1

      // once game is complete alert user with info
      if (this.isGameComplete()) {
        const msg = [
          'Success!',
          '', // just to add space
          // get difficulty of game played
          `Difficulty: ${ this.levels[ this.difficulty ]}`,
          // get final time
          `Time: ${ this.formattedTime }`
        ]

        // show alert to user
        alert(msg.join('\n'))
        // create new puzzle
        this.generatePuzzle()
      }
    },

    // game error checking for duplicate values
    isCellInvalid (row, col, value) {
      if (!value) {
        return true
      }

      // checks for duplicate number in row && does not check selected cell
      for (let c = 0; c < 9; c += 1) {
        if (this.puzzle[row][c].value === value && c !== col) {
          return true
        }
      }

      // checks for duplicate value in column && does not check selected cell
      for (let r = 0; r < 9; r += 1) {
        if (this.puzzle[r][col].value === value && r != row) {
          return true
        }
      }

      //checks for duplicate value in 3x3 grid section
      // to find section cell is in
      const rowStart = Math.floor(row/3) * 3
      const colStart = Math.floor(col/3) * 3

      // start of section +3 for 3x3 grid
      for (let r = rowStart; r < rowStart + 3; r += 1) {
        for (let c = colStart; c < colStart + 3; c += 1) {
          // if there is a value == to entered value && that IS NOT the current cell
          if (this.puzzle[r][c].value === value && !(r === row && c === col)) {
            return true
          }
        }
      }
      return false
    },

    // checks for game finish by iterating all cells looking for invalid/empty entries
    isGameComplete () {
      for (let r = 0; r < 9; r += 1) {
        for (let c = 0; c < 9; c += 1) {
          if (this.isCellInvalid(r, c, this.puzzle[r][c].value)) {
            return false
          }
        }
      }

      return true
    }
  },
  // call generatePuzzle once app starts
  mounted () {
    this.generatePuzzle()
  }
}
</script>

<style scoped>
.sudoku {
  width: 100%;
  max-width: 420px;
  margin: 0.5rem auto;

  font-family: Arial, Helvetica, sans-serif;
}

/* space between buttons and puzzle grid */
.grid {
  width: calc(9 * 40px);
  margin: 0.5rem auto 1rem;
}

.row {
  /* flex handles auto positioning */
  display: flex;
  /* will expand height to tallest element, smaller elements will be vertically centered */
  align-items: center;
  /* the first element will be all the way right, last element left, all other elements evenly spaced between */
  justify-content: space-between;
}

.cell {
  display: block;
  width: 40px; 
  height: 40px;
  /* border is included into the box */
  box-sizing: border-box;
  border: 1px solid #bbb;

  font-size: 24px;
  /* to vertically center text */
  line-height: 40px;
  /* to horizontally align text */
  text-align: center;
  cursor: default;
}

/* adds larger border to 3x3 grids */
.cell.border-right { 
  border-right-width: 3px; 
}
.cell.border-bottom {
  border-bottom-width: 3px;
}

/* bold original numbers */
.cell.original {
  font-weight: bold;
}

/* if cell is not original */
.cell:not(.original) {
  cursor: pointer;
}

/* change active cell */
.cell.active {
  /* active is more important and will always show */
  background-color: #00c !important;
  color: #fff;
}

/* when input is wrong */
.cell.invalid {
  background-color: red;
  color: white;
}

.btn {
  width: 38px;
  height: 38px;
  font-size: 24px;
  cursor: pointer;
}

/* disable and change cursor when disabled */
.btn:disabled {
  cursor: not-allowed;
}
</style>
