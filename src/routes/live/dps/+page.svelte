<script lang="ts">
  import { onMount } from "svelte";
  import { commands, type PlayersWindow } from "$lib/bindings";
  import { getClassColor } from "$lib/utils.svelte";
  import { goto } from "$app/navigation";
  import { getCoreRowModel } from "@tanstack/table-core";
  import { createSvelteTable } from "$lib/svelte-table";
  import { dpsPlayersColumnDefs } from "$lib/table-info";
  import FlexRender from "$lib/svelte-table/flex-render.svelte";
  import { SETTINGS } from "$lib/settings-store";
  import AbbreviatedNumber from "$lib/components/abbreviated-number.svelte";

  onMount(() => {
    fetchData();
    const interval = setInterval(fetchData, 200);

    return () => clearInterval(interval);
  });

  let dpsPlayersWindow: PlayersWindow = $state({ playerRows: [], localPlayerUid: -1, topValue: 0 });

  async function fetchData() {
    if (SETTINGS.misc.state.testingMode) {
      dpsPlayersWindow = await commands.getTestPlayerWindow();
    } else if (SETTINGS.general.state.bossOnly) {
      dpsPlayersWindow = await commands.getDpsBossOnlyPlayerWindow();
    } else {
      dpsPlayersWindow = await commands.getDpsPlayerWindow();
    }
  }

  const dpsTable = createSvelteTable({
    get data() {
      return dpsPlayersWindow.playerRows;
    },
    columns: dpsPlayersColumnDefs,
    getCoreRowModel: getCoreRowModel(),
    state: {
      get columnVisibility() {
        return SETTINGS.live.dps.players.state;
      },
    },
    meta: {
      get localPlayerUid() {
        return dpsPlayersWindow.localPlayerUid;
      },
    },
  });

  let SETTINGS_YOUR_NAME = $derived(SETTINGS.general.state.showYourName);
  let SETTINGS_OTHERS_NAME = $derived(SETTINGS.general.state.showOthersName);

  const totalDamage = $derived(() =>
    dpsPlayersWindow.playerRows.reduce((sum, row) => sum + row.totalValue, 0)
  );
  const totalDps = $derived(() =>
    dpsPlayersWindow.playerRows.reduce((sum, row) => sum + row.valuePerSec, 0)
  );
  const topDps = $derived(() => dpsPlayersWindow.playerRows[0]?.valuePerSec ?? 0);
  const updateIntervalSeconds = 0.2;

  function openSkillBreakdown(uid: number) {
    goto(`/live/dps/skills?playerUid=${uid}`);
  }
</script>

<section class="live-panel" aria-label="Live damage leaderboard">
  <div class="live-panel__header">
    <div class="live-panel__title">
      <span>Real-time view</span>
      Damage leaderboard
    </div>
    <div class="live-panel__metrics">
      <span>Players<strong>{dpsPlayersWindow.playerRows.length}</strong></span>
      <span>Total DPS<strong><AbbreviatedNumber num={totalDps()} /></strong></span>
      <span>Total DMG<strong><AbbreviatedNumber num={totalDamage()} /></strong></span>
      <span>Top DPS<strong><AbbreviatedNumber num={topDps()} /></strong></span>
      <span>Refresh<strong>{updateIntervalSeconds}s</strong></span>
    </div>
  </div>

  <div class="live-panel__body">
    <div class="live-panel__scroll">
      <table class="live-table" aria-label="Damage leaderboard">
        <thead>
          {#each dpsTable.getHeaderGroups() as headerGroup (headerGroup.id)}
            <tr>
              {#each headerGroup.headers as header (header.id)}
                <th
                  scope="col"
                  class={`${header.column.columnDef.meta?.class ?? ""} ${header.column.id === "playerInfo" ? "player-head" : ""}`.trim()}
                >
                  <FlexRender content={header.column.columnDef.header ?? "UNKNOWN HEADER"} context={header.getContext()} />
                </th>
              {/each}
            </tr>
          {/each}
        </thead>
        <tbody>
          {#each dpsTable.getRowModel().rows as row (row.id)}
            {@const isYou = row.original.uid !== -1 && row.original.uid == dpsPlayersWindow.localPlayerUid}
            {@const className = isYou
              ? (SETTINGS_YOUR_NAME !== "Hide Your Name" ? row.original.className : "Hidden Class")
              : SETTINGS_OTHERS_NAME !== "Hide Others' Name" ? row.original.className : "Hidden Class"}
            {@const fillWidth = dpsPlayersWindow.topValue > 0
              ? Math.min(100, (row.original.totalValue / dpsPlayersWindow.topValue) * 100)
              : 0}
            <tr
              class={isYou ? "is-you" : ""}
              role="button"
              tabindex="0"
              style={`--bar-color: ${getClassColor(className)}; --fill: ${fillWidth}%;`}
              onclick={() => openSkillBreakdown(row.original.uid)}
              onkeydown={(event) => {
                if (event.key === "Enter" || event.key === " ") {
                  event.preventDefault();
                  openSkillBreakdown(row.original.uid);
                }
              }}
            >
              {#each row.getVisibleCells() as cell (cell.id)}
                <td class={`${cell.column.columnDef.meta?.class ?? ""} ${cell.column.id === "playerInfo" ? "player-cell" : ""}`.trim()}>
                  <FlexRender content={cell.column.columnDef.cell ?? "UNKNOWN CELL"} context={cell.getContext()} />
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</section>
