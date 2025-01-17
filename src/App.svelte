<script lang="ts">
  import confetti from 'https://cdn.skypack.dev/canvas-confetti';

  const pins = 5
  const rows = 12

  const colors = ['black', 'white', 'deepskyblue', 'limegreen', '#8968e3', 'red', 'orange', 'yellow']
  let activeColor = ''

  let board = Array(rows).fill(0).map(() => Array(pins).fill(''))

  let secret = Array(pins).fill(0).map(() => colors[Math.ceil(Math.random() * colors.length) - 1])
  let secretReveal = Array(pins).fill(false)

  let rowScores = Array(rows).fill(undefined)

  function put(r: number, i: number) {
    board[r][i] = activeColor
  }

  function filled(row: string[]) {
    return row.every(p => p)
  }

  function calculateScore(r: number) {
    const row = [...board[r]];
    let correctPosition = 0, correctColor = 0
    const tempSecret = [...secret]
    secret.forEach((c, i) => {
      if (row[i] == c) {
        correctPosition++
        tempSecret[i] = 'x'
        row[i] = 'y'
      }
    })
    row.forEach((c, i) => {
      const j = tempSecret.indexOf(row[i])
      if (j >= 0) {
        correctColor++
        tempSecret[j] = ''
      }
    })
    rowScores[r] = {correctPosition, correctColor}
    if (correctPosition == pins) {
      confetti()
      secretReveal = secretReveal.fill(true)
    }
  }
</script>

<main>
  <h1>Logika / Mastermind</h1>

  <div class="game">
    <div class="colors">
      {#each colors as color}
        <div class="pin" class:active={activeColor == color}
             style="color: {color}"
             on:click={() => activeColor = color}>⬤</div>
      {/each}
    </div>

    <div class="board">
      <div class="row" style="margin-bottom: 2rem">
        {#each secret as c, i}
          <div class="pin" on:click={() => secretReveal[i] = !secretReveal[i]}
               style="color: {secretReveal[i] ? c : ''}">
            {secretReveal[i] ? '⬤' : '?'}
          </div>
        {/each}
      </div>

      {#each board as rows, r}
        <div class="row">
          {#each rows as c, i}
            <div class="pin" style="color: {c}"
                 on:click={() => put(r, i)}>{c ? '⬤' : '\u00a0'}</div>
          {/each}
        </div>
      {/each}
    </div>

    <div class="scores">
      {#each rowScores as score, r}
        <div class="row">
          {#if filled(board[r]) && !rowScores[r]}
            <button class="ready" on:click={() => calculateScore(r)}>✓</button>
          {/if}
          {#if score}
            {#each Array(score.correctPosition).fill(0) as _}
              <div class="score" style="color: black" title="One correct color and position">⬤</div>
            {/each}
            {#each Array(score.correctColor).fill(0) as _}
              <div class="score" style="color: white" title="One correct color, but wrong position">⬤</div>
            {/each}
          {:else}
            <div class="score"/>
          {/if}
        </div>
      {/each}
    </div>
  </div>
</main>

<style>
  .game {
    display: flex;
    gap: 2rem;
    user-select: none;
  }

  .board {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .colors, .scores {
    padding-top: 5rem;
  }

  .row {
    display: flex;
    flex-direction: row;
    height: 3rem;
  }

  .pin {
    border: 1px solid #ccc;
    margin: -1px 0 0 -1px;
    width: 3rem;
    line-height: 3rem;
  }

  .pin:hover {
    background: #cdcdcd;
    cursor: pointer;
  }

  .pin.active {
    background: #ccc;
    border-color: black;
    position: relative;
    z-index: 1;
  }

  .scores {
    width: calc(5 * 1.5rem);
  }

  .score {
    border-color: transparent;
    font-size: 0.75rem;
    width: 1.5rem;
    line-height: 3rem;
  }

  button.ready {
    background: #23ad23;
    color: white;
    font-size: 1.5rem;
    line-height: 0.5;
  }
</style>
