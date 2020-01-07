<template>
  <div>
    <div class="flex justify-center">
      <div class="flex font-mono text-2xl flex-col">
        <div class="flex" v-for="row in grid">
          <span
            :class="{'bg-green-500': showSolution && belongsToSolution}"
            class="uppercase border font-bold px-6 py-2"
            v-for="({letter, belongsToSolution}) in row"
          >
            {{ letter }}
          </span>
        </div>
      </div>
      <div class="flex items-center ml-16">
        <ul>
          <li class="text-2xl mt-4 uppercase" v-for="word in words">
            {{ word }}
          </li>
        </ul>
      </div>
    </div>
    <button @click="reroll" class="border-2 px-4 py-2 bg-gray-300">
      Reroll!
    </button>
    <button @click="showSolution = !showSolution" class="border-2 px-4 py-2 bg-gray-300">
      Show Solution!
    </button>
  </div>
</template>

<script>
import { ref } from '@vue/composition-api'

const lettersAndFrequency = {
  'a': 6.516,
  'b': 1.886,
  'c': 2.732,
  'd': 5.076,
  'e': 16.396,
  'f': 1.656,
  'g': 3.009,
  'h': 4.577,
  'i': 6.55,
  'j': 0.268,
  'k': 1.417,
  'l': 3.437,
  'm': 2.534,
  'n': 9.776,
  'o': 2.594,
  'p': 0.67,
  'q': 0.018,
  'r': 7.003,
  's': 7.27,
  't': 6.154,
  'u': 4.166,
  'v': 0.846,
  'w': 1.921,
  'x': 0.034,
  'y': 0.039,
  'z': 1.134,
  'ä': 0.578,
  'ö': 0.443,
  'ü': 0.995
}
const randomInBoundaries = (min, max) => Math.random() * (max - min) + min
const randomIntInBoundaries = (...args) => Math.round(randomInBoundaries(...args))

const getRandomItem = weightedObject => () => {
  const values = Object.keys(weightedObject)
  const weights = Object.values(weightedObject)
  const totalWeight = weights.reduce((acc, cur) => acc + cur)

  const randomNumber = randomInBoundaries(0, totalWeight)
  let weightedSum = 0

  for (let i = 0; i < values.length; i++) {
    weightedSum += weights[i]
    weightedSum = +weightedSum.toFixed(3)

    if (randomNumber <= weightedSum) {
      return values[i]
    }
  }
}

const getRandomLetter = getRandomItem(lettersAndFrequency)

const createEmptyGrid = size => Array.from(
  { length: size },
  () => Array.from({ length: size }, () => ({ letter: '-' }))
)
const fillGrid = grid => grid.map(row => row.map(cell => cell.belongsToSolution ? cell : {
  ...cell,
  letter: getRandomLetter()
}))

export default {
  props: {
    gridSize: {
      type: Number,
      default: 16
    },
    words: {
      type: Array,
      default: () => ['Adventskalender', 'Adventskalender', 'Adventskalender', 'Adventskalender', 'Kerze', 'Tannenbaum', 'Geschenk', 'Rute', 'Plätzchen', 'Familie', 'Schnee', 'Gans']
    }
  },
  setup (props) {
    const grid = ref(createEmptyGrid(props.gridSize))

    function insert (word, shouldInsertVertically = null, usedTries = 0) {
      if (word.length > props.gridSize) {
        return
      }

      const randomBeginning = randomIntInBoundaries(0, props.gridSize - word.length)
      const startIndex = randomIntInBoundaries(0, props.gridSize - word.length)
      const letters = Array.from(word)

      const willInsertVertically = shouldInsertVertically === null ? (Math.random() > 0.5) : shouldInsertVertically

      const isBlockedVertically = (letter, offset) => Boolean(grid.value[startIndex][randomBeginning + offset].belongsToSolution)
      const isBlockedHorizontally = (letter, offset) => Boolean(grid.value[startIndex + offset][randomBeginning].belongsToSolution)

      const isBlocked = letters.some(willInsertVertically ? isBlockedVertically : isBlockedHorizontally)

      if (isBlocked) {
        const triesNeededToChangeDirection = props.gridSize * 4
        const triesNeededToRerollGrid = triesNeededToChangeDirection * 2
        const shouldRerollGrid = usedTries >= triesNeededToRerollGrid

        const changeInsertionDirection = usedTries === triesNeededToChangeDirection
        const direction = changeInsertionDirection ? !shouldInsertVertically : shouldInsertVertically

        if (shouldRerollGrid) {
          reroll()
        }

        return insert(word, direction, usedTries + 1)
      }

      const insertVertically = (letter, offset) => {
        grid.value[startIndex][randomBeginning + offset] = {
          letter,
          belongsToSolution: true
        }
      }
      const insertHorizontally = (letter, offset) => {
        grid.value[startIndex + offset][randomBeginning] = {
          letter,
          belongsToSolution: true
        }
      }

      letters.forEach(willInsertVertically ? insertVertically : insertHorizontally)
    }

    function roll () {
      props.words.forEach(w => insert(w))
      grid.value = fillGrid(grid.value)
    }

    function reroll () {
      grid.value = createEmptyGrid(props.gridSize)
      roll()
    }

    roll()

    const showSolution = ref(false)

    return {
      grid,
      reroll,
      showSolution
    }
  }
}
</script>
