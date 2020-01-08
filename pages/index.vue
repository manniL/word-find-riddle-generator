<template>
  <div class="flex h-screen justify-center items-center">
    <WordSearchQuiz :grid-size="12" :words="wordsToHide" />
  </div>
</template>

<script>
import WordSearchQuiz from '~/components/WordSearchQuiz'
import { wordsToHide } from '@/assets/data'

export default {
  components: {
    WordSearchQuiz
  },
  setup () {
    return {
      wordsToHide
    }
  }
}
</script>
