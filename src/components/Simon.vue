<template>
  <div class="simon">
    <div class="simon__button" @click="startGame" :class="{fullSize: playing}">{{ centerButton }}</div>
    <div class="simon__body" :class="{pow: playing}">
      <div class="body__slice" v-show="playing" @click="clicked(1)" :class="{lit: isLit[1]}"></div>
      <div class="body__slice" v-show="playing" @click="clicked(2)" :class="{lit: isLit[2]}"></div>
      <div class="body__slice" v-show="playing" @click="clicked(3)" :class="{lit: isLit[3]}"></div>
      <div class="body__slice" v-show="playing" @click="clicked(4)" :class="{lit: isLit[4]}"></div>
      <div class="body__score" v-show="playing">{{ showScore }}</div>
    </div>
    <div class="simon__difficult" v-show="playing" :class="{pow: playing}">
      <p class="difficult__text">Mode:</p>

      <div
        class="difficult__button-body"
        @click="difficult(difficultList.easy)"
        :class="{active: difficultList.easy.selected}"
      >
        <p
          class="difficult__text difficult__text_button"
          :class="{active: difficultList.easy.selected}"
        >Easy</p>
        <button class="difficult__button" :class="{active: difficultList.easy.selected}"></button>
      </div>

      <div
        class="difficult__button-body"
        @click="difficult(difficultList.normal)"
        :class="{active: difficultList.normal.selected}"
      >
        <p
          class="difficult__text difficult__text_button"
          :class="{active: difficultList.normal.selected}"
        >Normal</p>
        <button class="difficult__button" :class="{active: difficultList.normal.selected}"></button>
      </div>

      <div
        class="difficult__button-body"
        @click="difficult(difficultList.hard)"
        :class="{active: difficultList.hard.selected}"
      >
        <p class="difficult__text difficult__text_button">Hard</p>
        <button class="difficult__button" :class="{active: difficultList.hard.selected}"></button>
      </div>
    </div>
  </div>
</template>

<script>
// Подключаем стороннию аудио библиотеку
import { Howl } from "howler";

export default {
  name: "app",
  data() {
    return {
      centerButton: "START",
      playing: false,
      isClickable: false,
      difficultList: {
        easy: { ms: 1500, selected: true },
        normal: { ms: 1000, selected: false },
        hard: { ms: 400, selected: false },
      },
      isWon: false,
      isWrong: false,
      score: 0,
      sequence: [],
      sequenceInterval: null,
      playerSequence: [],
      sounds: [
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"],
        }),
      ],
      isLit: {
        1: false,
        2: false,
        3: false,
        4: false,
      },
    };
  },
  computed: {
    showScore() {
      return this.score;
    },
  },
  methods: {
    // Устанавливаем сложность в зависимости от выбора пользователя
    difficult(selectDifficult) {
      if (selectDifficult.selected == false) {
        this.startGame();
      }
      this.difficultList.hard.selected = false;
      this.difficultList.normal.selected = false;
      this.difficultList.easy.selected = false;
      selectDifficult.selected = true;
      return (this.currentDifficult = selectDifficult.ms);
    },
    startGame() {
      this.playing = true;
      this.sequence = [];
      this.playerSequence = [];
      this.centerButton = "RESET";
      this.isWon = false;
      this.isWrong = false;
      this.score = 0;
      if (this.currentDifficult == undefined) {
        this.currentDifficult = 1500;
      }
      clearInterval(this.sequenceInterval);
      this.showSequence();
    },
    // Обработка клика
    clicked(slice) {
      if (this.isClickable) {
        this.playSound(slice);
        this.lightUp(slice);
        this.playerSequence.push(slice);
        this.checkWinLose();
      }
    },
    // Обработка хода пользователя
    checkWinLose() {
      for (let i = 0; i < this.playerSequence.length; i++) {
        // Проиграл :(
        if (this.playerSequence[i] !== this.sequence[i]) {
          this.centerButton = "Wrong!";
          this.isWrong = true;
          this.isClickable = false;
          return setTimeout(() => {
            this.startGame();
          }, 300);
        }
      }
      // Продолжаем...
      if (this.playerSequence.length === this.sequence.length) {
        this.playerSequence = [];
        this.score++;
        // Победа :)
        if (this.score === 20) {
          this.centerButton = "Winner!";
          this.isClickable = false;
          this.isWon = true;
        } else {
          setTimeout(() => {
            this.showSequence();
          }, 500);
        }
      }
    },
    // Событие загарает выбранную кнопку
    lightUp(slice) {
      this.isLit[slice] = true;
      setTimeout(() => {
        this.isLit[slice] = false;
      }, 300);
    },
    // Событие проигрывает звук выбранной кнопки
    playSound(slice) {
      this.sounds[slice - 1].play();
    },
    // Метод предназначет для расчета следующего хода
    showSequence(redo) {
      this.centerButton = "WAIT";
      let currentIndex = 0;
      // Если ход первый, то задержка 1 секунда, а иначе установленная сложность
      let speed = this.sequence.length === 0 ? 1000 : this.currentDifficult;
      this.isClickable = false;
      // Проверка если пользователь ошибся то добавление к старому массиву не происходит
      if (!redo) {
        this.sequence.push(Math.floor(Math.random() * 4 + 1));
      }
      // Ход программы
      this.sequenceInterval = setInterval(() => {
        if (currentIndex >= this.sequence.length) {
          clearInterval(this.sequenceInterval);
          // Далее очередь пользователя
          this.centerButton = "RESET";
          return (this.isClickable = true);
        }
        this.playSound(this.sequence[currentIndex]);
        this.lightUp(this.sequence[currentIndex]);
        currentIndex++;
      }, speed);
    },
  },
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Comfortaa:wght@300&display=swap");

$bg-color: #091921;
$red: #ff2020;
$white: #fff;
$yellow: #f5ff2d;
$green: #26ff6f;
$blue: #00b7ff;

@mixin center-align {
  align-items: center;
  display: flex;
  justify-content: center;
}

body {
  @include center-align();
  background: $bg-color;
  font-family: "Comfortaa", cursive;
  font-weight: 900;
  font-size: 20px;
  letter-spacing: 4px;
  overflow: hidden;
  margin: 0;
  -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
  -webkit-tap-highlight-color: transparent;
}

.simon {
  position: relative;
  display: flex;
  height: 100vh;
  width: 100vw;
  opacity: 1;
  overflow: hidden;

  .simon__button {
    @include center-align();
    align-content: center;
    flex-wrap: wrap;
    background: $bg-color;
    cursor: pointer;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    width: 90px;
    height: 90px;
    border: 4px solid $bg-color;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
      0 15px 15px rgba(0, 0, 0, 0.5);
    color: $white;
    transition: all 0.3s linear;
    z-index: 10;

    &:hover {
      box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
        inset 0 0px 15px rgba(255, 255, 255, 0.05),
        0 15px 15px rgba(0, 0, 0, 0.5), inset 0 0px 15px rgba(0, 0, 0, 0.5);
      text-shadow: 0px 0px 10px rgba(255, 255, 255, 0.8);
    }
    &:active {
      box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
        inset 0 -15px 15px rgba(255, 255, 255, 0.05),
        0 15px 15px rgba(0, 0, 0, 0.5), inset 0 15px 15px rgba(0, 0, 0, 0.5);
    }
  }

  .simon__body {
    position: relative;
    margin: auto;
    height: 300px;
    width: 300px;
    display: flex;
    flex-wrap: wrap;
    border-radius: 50%;
    padding: 8px;
    background: $bg-color;
    border: 4px solid $bg-color;
    transition: all 1s cubic-bezier(1, -0.08, 0.5, 0.5);
    box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
      inset 0 -15px 15px rgba(255, 255, 255, 0.05),
      0 15px 15px rgba(0, 0, 0, 0.5), inset 0 15px 15px rgba(0, 0, 0, 0.5);
    opacity: 0;
  }

  .body__score {
    @include center-align();
    position: absolute;
    right: 0;
    bottom: 0;
    left: 1px;
    top: 0px;
    width: 34px;
    height: 24px;
    margin: auto;
    font-size: 26px;
    text-align: center;
    color: $white;
    text-shadow: 0px 0px 5px rgba(255, 255, 255, 0.5);
    transition: opacity 0.3s linear;
  }

  .body__slice {
    margin: auto;
    cursor: pointer;
    height: 147px;
    width: 147px;
    transition: box-shadow 0.1s linear, opacity 1s linear;
    opacity: 1;
    &:nth-of-type(1) {
      background: $blue;
      border-top-left-radius: 100%;
      border-bottom-right-radius: 30%;
      box-shadow: 0px 0px 0px -3px $blue, 0px 0px 0px -20px $blue,
        0px 0px 0px -40px $blue, inset 0px 30px 20px -10px rgba(9, 25, 33, 0.5),
        inset 0px 0px 80px 0px rgba(9, 25, 33, 0.3);
      &.lit {
        box-shadow: -1px -1px 5px -3px $blue, -10px -10px 40px -20px $blue,
          -15px -15px 80px -40px $blue,
          inset 0px 30px 20px -10px rgba(9, 25, 33, 0);
      }
    }
    &:nth-of-type(2) {
      background: $green;
      border-top-right-radius: 100%;
      border-bottom-left-radius: 30%;
      box-shadow: 0px 0px 0px -3px $green, 0px 0px 0px -20px $green,
        0px 0px 0px -40px $green, inset 0px 30px 20px -10px rgba(9, 25, 33, 0.5),
        inset 0px 0px 80px 0px rgba(9, 25, 33, 0.3);
      &.lit {
        box-shadow: 1px -1px 5px -3px $green, 10px -10px 40px -20px $green,
          15px -15px 80px -40px $green,
          inset 0px 30px 20px -10px rgba(9, 25, 33, 0);
      }
    }
    &:nth-of-type(3) {
      background: $yellow;
      border-top-right-radius: 30%;
      border-bottom-left-radius: 100%;
      box-shadow: 0px 0px 0px -3px $yellow, 0px 0px 0px -20px $yellow,
        0px 0px 0px -40px $yellow,
        inset 0px 20px 10px -10px rgba(9, 25, 33, 0.5),
        inset 0px 0px 80px 0px rgba(9, 25, 33, 0.3);
      &.lit {
        box-shadow: -1px 1px 5px -3px $yellow, -10px 10px 40px -20px $yellow,
          -15px 15px 80px -40px $yellow,
          inset 0px 20px 10px -10px rgba(9, 25, 33, 0);
      }
    }
    &:nth-of-type(4) {
      background: $red;
      border-top-left-radius: 30%;
      border-bottom-right-radius: 100%;
      box-shadow: 0px 0px 0px -3px $red, 0px 0px 0px -20px $red,
        0px 0px 0px -40px $red, inset 0px 20px 10px -10px rgba(9, 25, 33, 0.4),
        inset 0px 0px 80px 0px rgba(9, 25, 33, 0.3);
      &.lit {
        box-shadow: 1px 1px 5px -3px $red, 10px 10px 40px -20px $red,
          15px 15px 80px -40px $red,
          inset 0px 20px 10px -10px rgba(9, 25, 33, 0);
      }
    }
  }
  .simon__difficult {
    position: absolute;
    right: 0;
    bottom: 100px;
    left: 550px;
    margin: auto;
    width: 200px;
    padding: 20px 20px 30px;
    background-color: $bg-color;
    border: 4px solid $bg-color;
    border-radius: 50px;
    opacity: 0;
    box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
      inset 0 -15px 15px rgba(255, 255, 255, 0.05),
      0 15px 15px rgba(0, 0, 0, 0.5), inset 0 15px 15px rgba(0, 0, 0, 0.5);

    .difficult__text {
      position: relative;
      font-size: 20px;
      margin: 0;
      color: $white;
      text-align: center;
      &.difficult__text_button {
        position: absolute;
        top: 10px;
        left: 0;
        right: 0;
        margin: auto;
        font-size: 14px;
        letter-spacing: 3px;
        &.active {
          color: $bg-color;
        }
      }
    }
    .difficult__button-body {
      cursor: pointer;
      position: relative;
      height: 30px;
      border: 4px solid $bg-color;
      border-radius: 30px;
      margin: 10px 0;
      box-shadow: 0 -15px 15px rgba(255, 255, 255, 0.05),
        inset 0 -15px 15px rgba(255, 255, 255, 0.05),
        0 15px 15px rgba(0, 0, 0, 0.5), inset 0 15px 15px rgba(0, 0, 0, 0.5);
      transition: 0.3s linear;

      .difficult__button {
        cursor: pointer;
        position: absolute;
        border: 1px solid $white;
        border-radius: 50%;
        background-color: $bg-color;
        outline: none;
        width: 30px;
        height: 30px;
        z-index: 1;
        transition: 0.3s linear;
        right: 160px;
        &.active {
          right: 0;
        }
      }
      &.active {
        background-color: $green;
        &:nth-of-type(2) {
          background-color: $yellow;
        }
        &:nth-of-type(3) {
          background-color: $red;
        }
      }
    }
  }
}

.fullSize {
  animation: fullSize 1s ease forwards;
}

.pow {
  animation: pow 1s ease forwards;
}

@keyframes fullSize {
  0% {
    left: 0;
    bottom: 0;
    transform: scale(1);
  }
  25% {
    transform: scale(0);
  }
  35% {
    left: 0;
    bottom: 0;
    opacity: 0;
  }
  50% {
    transform: scale(0);
    left: 400px;
    bottom: 400px;
    opacity: 1;
  }
  65% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
    left: 400px;
    bottom: 400px;
  }
}

@keyframes pow {
  0% {
    opacity: 0;
  }
  35% {
    opacity: 0;
  }
  50% {
    opacity: 1;
    transform: scale(0);
  }
  65% {
    transform: scale(1.1);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@media (max-width: 768px) {
  @keyframes fullSize {
    0% {
      transform: scale(1);
    }
    25% {
      transform: scale(0);
    }
    35% {
      top: 0;
      bottom: 0;
      opacity: 0;
    }
    50% {
      transform: scale(0);
      top: 20px;
      bottom: auto;
      opacity: 1;
    }
    65% {
      transform: scale(1.3);
    }
    100% {
      transform: scale(1);
      top: 20px;
      bottom: auto;
    }
  }
  .simon {
    width: 540px;
    margin: auto;
    overflow-y: initial;
    flex-wrap: wrap;
    .simon__body {
      margin: 160px auto 0;
      height: 270px;
      width: 270px;

      .body__slice {
        height: 130px;
        width: 130px;
      }
    }
    .simon__button {
      top: 0;
      bottom: 0;
    }
    .simon__difficult {
      position: relative;
      height: 180px;
      margin: 30px auto 0;
      top: 0;
      left: 0;
      bottom: 0;
    }
  }
}

@media (max-width: 540px) {
  .simon {
    width: auto;
  }
}
</style>
