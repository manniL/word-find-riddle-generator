<template>
  <div>
    <div class="flex justify-center">
      <div class="flex font-mono text-2xl flex-col">
        <div v-for="row in grid" class="flex">
          <span
            :class="{'bg-green-500': showSolution && belongsToSolution}"
            v-for="({letter, belongsToSolution}) in row"
            class="uppercase border font-bold px-6 py-2"
          >
            {{ letter }}
          </span>
        </div>
      </div>
      <div class="flex items-center ml-16">
        <ul>
          <li v-for="word in words" class="text-2xl mt-4 uppercase">
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
import { englishLetterFrequency } from '@/assets/data'

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

export default {
  props: {
    gridSize: {
      type: Number,
      default: 16
    },
    words: {
      type: Array,
      required: true
    },
    frequency: {
      type: Object,
      default: () => englishLetterFrequency
    }
  },
  setup (props) {
    const getRandomLetter = getRandomItem(props.frequency)

    const createEmptyGrid = size => Array.from(
      { length: size },
      () => Array.from({ length: size }, () => ({ letter: '-' }))
    )
    const fillGrid = grid => grid.map(row => row.map(cell => cell.belongsToSolution ? cell : {
      ...cell,
      letter: getRandomLetter()
    }))

    const grid = ref(createEmptyGrid(props.gridSize))

    function insert (word, shouldInsertVertically = null, usedTries = 0) {
      if (word.length > props.gridSize) {
        return
      }

      const randomBeginning = randomIntInBoundaries(0, props.gridSize - word.length)
      const startIndex = randomIntInBoundaries(0, props.gridSize - word.length)
      const letters = Array.from(word)

      const willInsertVertically = Math.random() > 0.5

      const isBlockedVertically = (letter, offset) => Boolean(grid.value[startIndex][randomBeginning + offset].belongsToSolution)
      const isBlockedHorizontally = (letter, offset) => Boolean(grid.value[startIndex + offset][randomBeginning].belongsToSolution)

      const isBlocked = letters.some(willInsertVertically ? isBlockedVertically : isBlockedHorizontally)

      if (isBlocked) {
        const triesNeededToChangeDirection = props.gridSize * 4

        const changeInsertionDirection = usedTries === triesNeededToChangeDirection
        const direction = changeInsertionDirection ? !shouldInsertVertically : shouldInsertVertically

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
      return true
    }

    function roll () {
      const sortedWords = props.words.slice().sort((a, b) => b.length - a.length)
      while (true) {
        const hasPlacedAllWords = sortedWords.every(w => insert(w))
        if (!hasPlacedAllWords) {
          grid.value = createEmptyGrid(props.gridSize)
          continue
        }
        grid.value = fillGrid(grid.value)
        break
      }
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
