<template>
  <div class="container">
    <div class="p-5 mb-4 bg-light rounded-3">
      <div class="container-fluid">
        <h1 class="display-5 fw-bold">Hızlı Yazma Yarışması</h1>
        <p class="col-md-8 fs-4">Ne kadar hızlı olduğunu test et !</p>
        <div v-if="isFinish" class="alert alert-danger">
            <h1>Yarışma Bitti !</h1>
            <div class="d-flex justify-content-between w-25">
                <div>
            <!--<h4><span class="badge bg-success"> {{dks}} DKS</span></h4>
            <h4><span class="badge bg-success">Doğruluk Yüzdesi: {{truePercent}} %</span></h4>-->
            </div>
            <h4><span class="badge bg-success">Doğru sayısı: {{lastTrueCount}}</span></h4>
            <h4><span class="badge bg-danger ms-2">Yanlış sayısı: {{falseCount}}</span></h4>
            </div>
            <button class="btn btn-dark" type="button" @click="resetAll">Yenile</button>
        </div>
        <div v-else>
        <div class="d-flex justify-content-between w-25">
            <h4><span class="badge bg-success">Doğru sayısı: {{trueCount}}</span></h4>
            <h4><span class="badge bg-danger ms-2">Yanlış sayısı: {{falseCount}}</span></h4>
        </div>
        <div class="card">
          <div class="card-body">
            <span
              v-for="(word, key) in words"
              :key="key"
              :class="[(key != this.word_i || writingWordControl),lastWordControl(key)]"
              class="mx-1 words"
            >
              {{ word }}
            </span>
          </div>
        </div>
        <div class="card mt-1 bg-primary">
          <div class="card-body">
            <div class="input-group input-group-lg">
              <input type="text" class="form-control" v-model="writingWord" />
              <button class="btn btn-dark" disabled type="button">{{timer}} sn.</button>
              <button :disabled="isRunning" class="btn btn-dark" type="button" @click="resetAll()">Yenile</button>
            </div>
          </div>
        </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import wordList from '@/assets/wordList.json'

export default {
  data () {
    return {
      words: null,
      writingWord: null,
      isTrue: true,
      isWordTrueIndexes: [],
      isWordFalseIndexes: [],
      trueCount: 0,
      falseCount: 0,
      word_i: 0,
      timer: 60,
      interval: false,
      isRunning: false,
      isFinish: false,
      lastTrueCount: 0,
      lastFalseCount: 0,
      wordList: wordList,
      startListIndex: 0,
      endListIndex: 20
    }
  },
  watch: {
    writingWord (val) {
      if (!val || val === ' ') {
        this.writingWord = ''
        return
      }
      if (!this.isRunning) {
        this.toggleTimer()
      }
      const lengthWords = this.words.length
      if ((this.word_i) === lengthWords) {
        this.isWordTrueIndexes = []
        this.isWordFalseIndexes = []
        this.word_i = 0
      }
      const currentWord = this.words[this.word_i].slice(0, val.length)
      const userWord = val.replace(' ', '')
      this.isTrue = currentWord === userWord || typeof currentWord === 'undefined'

      if (val.indexOf(' ') !== -1) {
        if (this.isTrue) {
          this.trueCount++
          this.isWordTrueIndexes.push(this.word_i)
        } else {
          this.isWordFalseIndexes.push(this.word_i)
          this.falseCount++
        }
        this.writingWord = ''
        this.word_i++
        if (this.word_i === this.endListIndex) {
          this.startListIndex += this.startListIndex
          this.endListIndex += this.startListIndex
          this.isWordTrueIndexes = []
          this.isWordFalseIndexes = []
          this.getWords()
        }
      }
    }
  },
  computed: {
    writingWordControl () {
      return this.isTrue ? 'writing-word' : 'writing-word bg-danger'
    }
    // dks () {
    //  return (this.trueCount + this.falseCount) / this.timer
    // },
    // truePercent () {
    //  const percent = (100 / this.dks)
    //  return (percent * this.trueCount)
    // }
  },
  mounted () {
    this.shuffle()
    this.getWords()
  },
  methods: {
    lastWordControl (key) {
      return this.isWordTrueIndexes.includes(key) ? 'textTrue' : (this.isWordFalseIndexes.includes(key) ? 'textFalse' : '')
    },
    toggleTimer () {
      this.isRunning = true
      this.interval = setInterval(this.timeProcess, 1000)
    },
    timeProcess () {
      if (this.timer === 0) {
        clearInterval(this.interval)
        this.lastTrueCount = this.trueCount
        this.lastFalseCount = this.falseCount
        this.isFinish = true
        return
      }
      this.timer--
    },
    getWords () {
      this.words = this.wordList.splice(this.startListIndex, this.endListIndex)
    },
    resetAll () {
      this.words = null
      this.writingWord = null
      this.isTrue = true
      this.isWordTrueIndexes = []
      this.isWordFalseIndexes = []
      this.trueCount = 0
      this.falseCount = 0
      this.word_i = 0
      this.timer = 60
      this.interval = false
      this.isRunning = false
      this.isFinish = false
      this.lastTrueCount = 0
      this.lastFalseCount = 0
      this.wordList = wordList
      this.startListIndex = 0
      this.endListIndex = 20
      this.shuffle()
      this.getWords()
    },
    shuffle () {
      const numbers = [...this.wordList]
      let first
      let second
      let temp
      const count = numbers.length
      for (let i = count - 1; i > 0; i--) {
        first = Math.floor(Math.random() * (i + 1))
        second = Math.floor(Math.random() * (i + 1))
        temp = numbers[first]
        numbers[first] = numbers[second]
        numbers[second] = temp
      }
      this.wordList = numbers
    }
  }
}
</script>

<style scoped>
.words {
  font-size: 25px;
  padding: 5px;
  border-radius: 5px;
}
.textTrue {
    color:green;
}
.textFalse {
    color:red;
}
.writing-word {
  background-color: rgba(168, 168, 168, 0.712);
  color: black;
}
</style>
