<script lang="ts">
  const size = 10;

  enum Content {
    flag = "🚩",
    bomb = "💣",
    collision = "💥",
  }

  let _map = [...Array(size)].map(() => [...Array(size)].map(() => 0));
  const p = () => Math.floor(Math.random() * size);
  let bombs: Set<[number, number]> = new Set();
  let flag_map: Set<string> = new Set();
  let _safe_map: Set<String> = new Set();
  let board_map = _map.map((row) => row.map(() => ""));

  function generate() {
    console.log("generate...");
    flag_map.clear();
    _safe_map.clear();
    _map = [...Array(size)].map(() => [...Array(size)].map(() => 0));
    bombs = new Set([...Array(size)].map(() => [p(), p()]));
    bombs.forEach(([x, y]) => (_map[x][y] = -1));
    bombs.forEach(([x, y]) => {
      let _x = 0;
      let _y = 0;
      for (let i = -1; i < 2; i++) {
        _x = x + i;
        for (let j = -1; j < 2; j++) {
          _y = y + j;
          let v = _map?.[_x]?.[_y];
          if (v !== -1 && v !== undefined) _map[_x][_y]++;
        }
      }
    });
    _map.forEach((row, x) => {
      row.forEach((_, y) => {
        board_map[x][y] = "";
      });
    });
  }

  let _show = true;
  function show() {
    _map.forEach((row, x) =>
      row.forEach((val, y) => {
        val != -1
          ? (board_map[x][y] = _show ? val + "" : "")
          : (board_map[x][y] = _show ? Content.collision : "");
      })
    );
    _show = !_show;
  }

  function show_safe_cell(x: number, y: number) {
    let v = _map?.[x]?.[y];
    console.log(x, y);

    if (v >= 0 && v != undefined) {
      if (v != 0) {
        // 不是空白格退出递归
        board_map[x][y] = v + "";
        _safe_map.add(`${x}-${y}`);
        return;
      }
      board_map[x][y] = "0";

      let _x = 0;
      let _y = 0;
      for (let i = -1; i < 2; i++) {
        _x = x + i;
        for (let j = -1; j < 2; j++) {
          _y = y + j;
          let _v = _map?.[_x]?.[_y];
          if (_v == undefined) continue;
          if (_x == x && _y == y) continue;
          if (_safe_map.has(`${_x}-${_y}`)) continue;
          _safe_map.add(`${_x}-${_y}`);
          show_safe_cell(_x, _y);
        }
      }
    }
  }

  generate();
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
            const v = _map[x][y];
            if (v == -1) {
              alert("你踩雷了，游戏失败🥲。");
              show();
              return;
            }

            show_safe_cell(x, y);
          }}
          on:contextmenu={(e) => {
            e.preventDefault();

            let p = `[${x},${y}]`;
            let cell = board_map[x][y];
            if (cell == Content.flag) {
              board_map[x][y] = "";
              flag_map.delete(p);
            } else {
              board_map[x][y] = Content.flag;
              flag_map.add(p);
            }

            let win = true;
            bombs.forEach(([_x, _y]) => {
              if (!flag_map.has(`[${_x},${_y}]`)) {
                win = false;
              }
            });
            if (win) {
              alert("恭喜，赢得胜利✌！");
              flag_map.clear();
              show();
            }
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
