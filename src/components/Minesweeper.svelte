<script lang="ts">
  import { beforeUpdate } from 'svelte';
  import SmileyDead from '../icons/smiley-dead.svelte';
  import SmileyLaugh from '../icons/smiley-laugh.svelte';
  import SmileySunglasses from '../icons/smiley-sunglasses.svelte';

  const numberColors = {
    0: '',
    1: 'one',
    2: 'two',
    3: 'three',
    4: 'four',
    5: 'five',
    6: 'six',
    7: 'seven',
    8: 'eight',
  };

  enum GameStatus {
    RUNNING,
    PAUSED,
    GAME_OVER,
    VICTORY,
  }

  interface IPosition {
    x: number;
    y: number;
  }

  interface IMineField {
    isMine: boolean;
    isRevealed: boolean;
    isFlagged: boolean;
    mineCounter: number | null; // If its a mine the counter will be null
    position: IPosition;
  }

  export let rows: number;
  export let columns: number;
  export let mines: number;

  let mineField: IMineField[][];
  let gameStatus = GameStatus.PAUSED;

  let counterText = '000';
  let timerText = '000'; // TODO: add timer text

  beforeUpdate(() => {
    mineField = generateMineField(rows, columns, mines);
    counterText = getCounterText(mineField);
    console.log('mf', mineField);
  });

  function getCounterText(mineField) {
    if (!mineField) {
      return '000';
    }
    const flagCounter = mineField
      .reduce((acc: IMineField[], val: IMineField[]) => acc.concat(val), [])
      .filter((m: IMineField) => m.isFlagged && !m.isRevealed).length;
    return toDisplayNumber(mines - flagCounter);
  }

  function toMineField(x: number, y: number): IMineField {
    return {
      isMine: false,
      isFlagged: false,
      isRevealed: false,
      mineCounter: null, // If its a mine the counter will remain null
      position: { x, y },
    };
  }

  function createEmptyMineField(rows: number, columns: number): IMineField[][] {
    return Array(rows)
      .fill(0)
      .map((_, x) =>
        Array(columns)
          .fill(0)
          .map((_, y) => toMineField(x, y))
      );
  }

  function generateMineField(rows: number, columns: number, mines: number) {
    console.log(rows, columns, mines);
    return setMineCounters(
      placeMinesOnField(
        createEmptyMineField(rows, columns),
        rows,
        columns,
        mines
      )
    );
  }

  const isMine = (x: number, y: number): boolean => mineField[x][y].isMine;
  const getRandomNumber = (num: number): number =>
    Math.floor(Math.random() * num);
  const isFlagged = (x: number, y: number): boolean =>
    mineField[x][y].isFlagged;

  function revealAllMines() {
    mineField.map((row) =>
      row.map((field) => {
        if (field.isMine) {
          field.isRevealed = true;
          field.isFlagged = false;
        }
      })
    );
  }

  function revealCell(x: number, y: number) {
    if (!mineField) {
      return;
    }
    console.log('reveal ', x, y);

    mineField.map((row, i) =>
      row.map((field, j) => {
        console.log('reveal ', x, y, i, j);
        if (y === i && x === j) {
          field.isRevealed = true;
          field.isFlagged = false;
        }
      })
    );

    console.log('mf after reveal', mineField);

    // const field = mineField[x][y];
    // field.isRevealed = true;
    // field.isFlagged = false;
    const winningCondition =
      mineField
        .reduce((acc: IMineField[], val: IMineField[]) => acc.concat(val), [])
        .filter((m: IMineField) => !m.isMine && !m.isRevealed).length === 0;

    if (winningCondition) {
      // this.clearTimer();
      gameStatus = GameStatus.VICTORY;
    } else {
      revealSurroundingCells(x, y);
    }
  }

  const getPositionsOfSurroundingCells = (x: number, y: number) =>
    constructSurroundingCells(x, y);

  function revealSurroundingCells(x: number, y: number) {
    if (mineField[x][y].mineCounter !== 0) {
      return;
    }
    const cellPositions = getPositionsOfSurroundingCells(x, y);
    const surroundingMines = cellPositions
      .map(([posX, posY]) => mineField[posX][posY])
      .filter((mine: IMineField) => !mine.isRevealed);
    surroundingMines.forEach((mine: IMineField) => {
      revealCell(mine.position.x, mine.position.y);
    });
  }

  function placeMinesOnField(
    mineField: IMineField[][],
    rows: number,
    columns: number,
    numberOfMines: number
  ) {
    while (numberOfMines > 0) {
      const rowIdx = getRandomNumber(rows);
      const colIdx = getRandomNumber(columns);
      if (!mineField[rowIdx][colIdx].isMine) {
        mineField[rowIdx][colIdx].isMine = true;
        numberOfMines--;
      }
    }
    return mineField;
  }

  function setMineCounters(mineField: IMineField[][]): IMineField[][] {
    return mineField.map((row, x) =>
      row.map((field, y) => {
        return field.isMine
          ? field
          : {
              ...field,
              mineCounter: countSurroundingMines(x, y, mineField),
            };
      })
    );
  }

  function countSurroundingMines(
    x: number,
    y: number,
    mineField: IMineField[][]
  ): number {
    let counter = 0;
    const surroundingCells = constructSurroundingCells(x, y);
    surroundingCells.forEach((pos: number[]) => {
      if (mineField[pos[0]][pos[1]].isMine) {
        counter++;
      }
    });
    return counter;
  }

  function constructSurroundingCells(x: number, y: number) {
    return getSurroundingCells(x, y).reduce(
      (acc: Array<[number, number]>, val) =>
        val[0] < 0
          ? acc
          : val[1] < 0
          ? acc
          : val[0] >= rows
          ? acc
          : val[1] >= columns
          ? acc
          : [...acc, val],
      []
    );
  }

  function getSurroundingCells(x: number, y: number): Array<[number, number]> {
    return [
      [x - 1, y - 1],
      [x, y - 1],
      [x + 1, y - 1],
      [x - 1, y],
      [x + 1, y],
      [x - 1, y + 1],
      [x, y + 1],
      [x + 1, y + 1],
    ];
  }

  function toDisplayNumber(num: number) {
    if (num > 999) {
      return '999';
    }
    let displayNumber = '';
    if (num < 100) {
      displayNumber += '0';
    }
    if (num < 10) {
      displayNumber += '0';
    }
    return displayNumber.concat(String(num));
  }

  function onCellClick(x: number, y: number) {
    console.log('gs', gameStatus, x, y);
    if (
      gameStatus === GameStatus.VICTORY ||
      gameStatus === GameStatus.GAME_OVER ||
      isFlagged(x, y)
    ) {
      return;
    }
    if (gameStatus !== GameStatus.RUNNING) {
      gameStatus = GameStatus.RUNNING;
      // this.timerId = window.setInterval(this.updateTimer.bind(this), 1000);
    }
    isMine(x, y) ? gameOver() : revealCell(x, y);
  }

  function gameOver() {
    // this.clearTimer();
    gameStatus = GameStatus.GAME_OVER;
    revealAllMines();
  }

  console.log(mineField);
</script>

<div class="game-wrapper">
  <div class="mine-sweeper">
    <div class="game-state">
      <div class="counter"><span>{counterText}</span></div>
      <div
        class="game-status"
        onClick={() => this.resetGame()}
        title="Reset the game">
        {#if gameStatus === GameStatus.PAUSED || gameStatus === GameStatus.RUNNING}
          <SmileyLaugh />
        {:else if gameStatus === GameStatus.GAME_OVER}
          <SmileyDead />
        {:else if gameStatus === GameStatus.VICTORY}
          <SmileySunglasses />
        {/if}
      </div>
      <div class="timer"><span>{timerText}</span></div>
    </div>
    {#if mineField}
      {#each mineField as rows, i}
        <div class="grid-row">
          {#each rows as _, j}
            <div
              class="grid-cell {mineField[i][j].isRevealed && mineField[i][j].mineCounter ? numberColors[mineField[i][j].mineCounter] : ''}"
              class:revealed={mineField[i][j].isRevealed}
              class:mine={mineField[i][j].isRevealed}
              class:flagged={mineField[i][j].isFlagged}
              on:click={(_) => onCellClick(i, j)}>
              {(mineField[i][j].mineCounter && mineField[i][j].isRevealed && mineField[i][j].mineCounter) || ''}
            </div>
          {/each}
        </div>
      {/each}
    {/if}
  </div>
</div>
