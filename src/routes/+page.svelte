<script lang="ts">
  const size = 10;

  enum Content {
    flag = "🚩",
    bomb = "💣",
    collision = "💥",
  }

  let _board_map = [...Array(size)].map(() => [...Array(size)].map(() => 0));
  const p = () => Math.floor(Math.random() * size);
  let bombs: Set<[number, number]> = new Set();
  let flag_map: Set<string> = new Set();

  function generate() {
    console.log("generate...");
    _board_map = [...Array(size)].map(() => [...Array(size)].map(() => 0));
    bombs = new Set([...Array(5)].map(() => [p(), p()]));
    bombs.forEach(([x, y]) => (_board_map[x][y] = -1));
    bombs.forEach(([x, y]) => {
      let _x = 0;
      let _y = 0;
      for (let i = 0; i < 3; i++) {
        _x = x - 1 + i;
        if (_x < 0 || _x > 8) continue;
        for (let j = 0; j < 3; j++) {
          _y = y - 1 + j;
          if (_y > 8 || _y < 0) continue;
          let v = _board_map?.[_x]?.[_y];
          if (v !== -1 && v !== undefined) _board_map[_x][_y]++;
        }
      }
    });
  }

  generate();

  let board_map = _board_map.map((row) => row.map(() => ""));

  function show(content?: string) {
    _board_map.forEach((row, x) =>
      row.forEach((val, y) => {
        if (typeof content == "string") {
          board_map[x][y] = content;
          return;
        }
        val != -1
          ? (board_map[x][y] = val + "")
          : (board_map[x][y] = Content.collision);
      })
    );
  }
</script>

<svelte:head>
  <title>作业</title>
  <meta name="description" content="计算机软件课程设计作业" />
</svelte:head>

<div class="flex justify-center bg-white rounded-lg shadow-lg">
  <div class="board p-4">
    {#each board_map as row, x}
      {#each row as cell, y}
        <button
          class="w-10 h-10"
          on:click={(e) => {
            e.preventDefault();
            const v = _board_map[x][y];
            if (v == -1) {
              show();
              alert("游戏失败");
              return;
            }
            board_map[x][y] = v + "";
          }}
          on:contextmenu={(e) => {
            e.preventDefault();
            board_map[x][y] = Content.flag;
            flag_map.add(x+'-'+y);
            let win = true;
            bombs.forEach((v1) => {
              if (!flag_map.has(v1[0]+'-'+v1[1])) {
                win = false
              }
            });

            if (win) alert("你赢了！");
          }}
        >
          {cell}
        </button>
      {/each}
    {/each}
  </div>
</div>
<hr />
<div class="flex justify-center container bg-white rounded-lg shadow-lg">
  <button
    class="btn"
    on:click={() => {
      generate();
      show("");
    }}>重开</button
  >
  <button
    class="btn"
    on:click={() => {
      show();
    }}>作弊</button
  >
</div>

<style>
  .board {
    display: grid;
    gap: 0.75rem;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: repeat(10, 1fr);
    place-content: center;
  }

  .btn {
    height: 2rem;
    width: 6rem;
    margin: 0.5rem;
  }
</style>
