<template>
  <div class="game-scrin">
    <div class="game-board">
      <div class="row" v-for="(row, rowKey) in playAreaCells" :key="rowKey">
        <div
          class="cell"
          :class="{ 'selected': playAreaCells[rowKey][cellKey].selected }"
          v-for="(cell, cellKey) in row"
          :key="cellKey"
          @click="onClick(rowKey, cellKey)"
        >
          <span
            v-if="playAreaCells[rowKey][cellKey].selected"
            :class="markClass(rowKey, cellKey)"
          />
        </div>
      </div>
    </div>

    <div
      v-show="showEndGameBanner"
      class="banner-container"
    >
      <div class="overlay" />

      <div class="banner">
        <div class="title">
          {{ endGameMessage }}
        </div>

        <div
          class="retry-btn"
          @click="retry"
        >
          Retry
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

interface Cell {
  selected: boolean;
  value: null | string;
}

@Component
export default class Playground extends Vue {
  protected playAreaCells: Array<Array<Cell>> = [
    [{ selected: false, value: null }, { selected: false, value: null }, { selected: false, value: null }],
    [{ selected: false, value: null }, { selected: false, value: null }, { selected: false, value: null }],
    [{ selected: false, value: null }, { selected: false, value: null }, { selected: false, value: null }],
  ];

  protected move = 0;
  protected endGameMessage = '';
  protected showEndGameBanner = false;

  protected onClick(rowKey: number, cellKey: number): void {
    const currentCell = this.playAreaCells[rowKey][cellKey];

    if (currentCell.selected) return;

    currentCell.selected = true;
    currentCell.value = this.move % 2 === 0 ? 'cross' : 'zero';

    if (this.checkEnd('cross')) {
      this.$nextTick();
      this.endGameMessage = 'Cross win!';
      this.showEndGameBanner = true;
      return;
    }

    if (this.checkEnd('zero')) {
      this.$nextTick();
      this.endGameMessage = 'Zero win!';
      this.showEndGameBanner = true;
      return;
    }

    if (this.checkDeadHeat()) {
      this.$nextTick();
      this.endGameMessage = 'Dead heat';
      this.showEndGameBanner = true;
      return;
    }

    this.move += 1;
  }

  protected retry(): void {
    this.clearGameBoard();
    this.showEndGameBanner = false;
    this.endGameMessage = '';
  }

  protected clearGameBoard(): void {
    this.playAreaCells.forEach((row: Array<Cell>) => {
      row.forEach((cell: Cell) => {
        cell.selected = false;
        cell.value = null;
      });
    });
  }

  protected markClass(rowKey: number, cellKey: number): string | void {
    const currentCell = this.playAreaCells[rowKey][cellKey];

    if (currentCell.selected) {
      return currentCell.value === 'cross' ? 'cross' : 'zero';
    }

    return '';
  }

  protected checkEnd(player: string): boolean | void {
    const xy00 = this.playAreaCells[0][0].value;
    const xy01 = this.playAreaCells[0][1].value;
    const xy02 = this.playAreaCells[0][2].value;
    const xy10 = this.playAreaCells[1][0].value;
    const xy11 = this.playAreaCells[1][1].value;
    const xy12 = this.playAreaCells[1][2].value;
    const xy20 = this.playAreaCells[2][0].value;
    const xy21 = this.playAreaCells[2][1].value;
    const xy22 = this.playAreaCells[2][2].value;

    if (xy00 === player && xy01 === player && xy02 === player) return true;
    if (xy10 === player && xy11 === player && xy12 === player) return true;
    if (xy20 === player && xy21 === player && xy22 === player) return true;
    if (xy00 === player && xy10 === player && xy20 === player) return true;
    if (xy01 === player && xy11 === player && xy21 === player) return true;
    if (xy02 === player && xy12 === player && xy22 === player) return true;
    if (xy00 === player && xy11 === player && xy22 === player) return true;
    if (xy02 === player && xy11 === player && xy20 === player) return true;

    return false;
  }

  protected checkDeadHeat(): boolean {
    return this.playAreaCells.every((row: Array<Cell>) => row.every((cell: Cell) => cell.selected));
  }
}
</script>

<style scoped lang="scss">
.game-board {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  .row {
    display: flex;
  }

  .cell {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 80px;
    height: 80px;
    background-color: #bfcbff;
    border: 1px solid #4a5896;
    cursor: pointer;
    transition: .1s;
    position: relative;

    &:not(.selected):hover {
      background-color: #e1e6ff;
    }

    span.zero {
      height: 45px;
      width: 45px;
      border: 3px solid black;
      border-radius: 50%;
    }

    span.cross {
      position: relative;
      height: 50px;
      width: 50px;

      &:before, &:after {
        position: absolute;
        content: ' ';
        width: 2px;
        height: 55px;
        background-color: black;
      }

      &:before {
        transform: rotate(45deg);
      }

      &:after {
        transform: rotate(-45deg);
      }
    }
  }
}

.banner-container {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100vh;
  left: 0;
  top: 0;

  .banner {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
    width: 300px;
    height: 200px;
    background-color: white;
    border-radius: 5px;
    z-index: 999;

    .title {
      font-size: 24px;
      color: lightgreen;
    }

    .retry-btn {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100px;
      height: 50px;
      border-radius: 5px;
      background-color: lightskyblue;
      cursor: pointer;
    }
  }

  .overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: black;
    opacity: .6;

  }
}
</style>
