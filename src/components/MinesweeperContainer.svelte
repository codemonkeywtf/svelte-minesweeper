<script lang="ts">
  import Minesweeper from './Minesweeper.svelte';

  enum Difficulty {
    EASY,
    MEDIUM,
    HARD,
  }

  let difficulty = Difficulty.EASY;

  $: settings = setSettings(difficulty);

  function getDifficultySettings(
    difficulty: Difficulty
  ): { rows: number; columns: number; mines: number } {
    return {
      0: { rows: 8, columns: 8, mines: 10 },
      1: { rows: 16, columns: 16, mines: 40 },
      2: { rows: 16, columns: 30, mines: 99 },
    }[difficulty];
  }

  function setSettings(difficulty: Difficulty) {
    const { rows, columns, mines } = getDifficultySettings(difficulty);

    return {
      rows,
      columns,
      mines,
    };
  }
</script>

<div class="header">
  <h1>Minesweeper</h1>
  <div class="instructions">
    <span> To flag cells use <span class="hotkey">right click</span>. </span>
    <span>
      To perimeter check already revealed cells use
      <span class="hotkey">double click</span>.
    </span>
    <span>
      To reset the game press the
      <span class="hotkey">R</span>
      key or the{' '}
      <span class="hotkey">smiley</span>
      button.
    </span>
  </div>
</div>
<div class="difficulty">
  <h3>Difficulty</h3>
  <div class="difficulty-container">
    <span
      class={difficulty === Difficulty.EASY ? 'active' : ''}
      on:click={() => (difficulty = Difficulty.EASY)}>
      Easy
      <div class="difficulty-mine-container"><i class="difficulty-mine" /></div>
    </span>
    <span
      class={difficulty === Difficulty.MEDIUM ? 'active' : ''}
      on:click={() => (difficulty = Difficulty.MEDIUM)}>
      Medium
      <div class="difficulty-mine-container">
        <i class="difficulty-mine" />
        <i class="difficulty-mine" />
      </div>
    </span>
    <span
      class={difficulty === Difficulty.HARD ? 'active' : ''}
      on:click={() => (difficulty = Difficulty.HARD)}>
      Hard
      <div class="difficulty-mine-container">
        <i class="difficulty-mine" />
        <i class="difficulty-mine" />
        <i class="difficulty-mine" />
      </div>
    </span>
  </div>
</div>
<Minesweeper {...settings} />
