<script lang="ts">
  import { onMount } from "svelte";
  import { commands, type SkillsWindow } from "$lib/bindings";
  import { getClassColor } from "$lib/utils.svelte";
import { page } from "$app/state";
import { createSvelteTable, FlexRender } from "$lib/svelte-table";
import { dpsPlayersColumnDefs, dpsSkillsColumnDefs } from "$lib/table-info";
import { getCoreRowModel } from "@tanstack/table-core";
import { SETTINGS } from "$lib/settings-store";
import AbbreviatedNumber from "$lib/components/abbreviated-number.svelte";

  const playerUid: string = page.url.searchParams.get("playerUid") ?? "-1";

  onMount(() => {
    fetchData();
    const interval = setInterval(fetchData, 200);
    return () => clearInterval(interval);
  });

  let dpsSkillBreakdownWindow: SkillsWindow | undefined = $state(undefined);

  async function fetchData() {
    try {
      const result = SETTINGS.misc.state.testingMode ? await commands.getTestSkillWindow(playerUid) : await commands.getDpsSkillWindow(playerUid);
      if (result.status !== "ok") {
        console.warn("Failed to get skill window: ", result.error);
        return;
      } else {
        dpsSkillBreakdownWindow = result.data;
        // console.log("dpsSkillBreakdown: ", +Date.now(), $state.snapshot(dpsSkillBreakdownWindow));
      }
    } catch (e) {
      console.error("Error fetching data: ", e);
    }
  }

  const inspectedPlayerTable = createSvelteTable({
    get data() {
      if (dpsSkillBreakdownWindow !== undefined) {
        return [dpsSkillBreakdownWindow.inspectedPlayer];
      } else {
        return [];
      }
    },
    columns: dpsPlayersColumnDefs,
    getCoreRowModel: getCoreRowModel(),
    state: {
      get columnVisibility() {
        return SETTINGS.live.dps.skillBreakdown.state;
      },
    },
  });

const dpsSkillBreakdownTable = createSvelteTable({
  get data() {
    if (dpsSkillBreakdownWindow !== undefined) {
      return dpsSkillBreakdownWindow.skillRows;
    } else {
      return [];
    }
  },
  columns: dpsSkillsColumnDefs,
  getCoreRowModel: getCoreRowModel(),
  state: {
    get columnVisibility() {
      return SETTINGS.live.dps.skillBreakdown.state;
    },
  },
});

const skillCount = $derived(() => dpsSkillBreakdownWindow?.skillRows.length ?? 0);
const inspectedPlayer = $derived(() => dpsSkillBreakdownWindow?.inspectedPlayer ?? null);
const inspectedClassColor = $derived(() => {
  const player = inspectedPlayer();
  if (!player) {
    return "transparent";
  }
  return getClassColor(player.className ?? "Hidden Class");
});
const topSkillName = $derived(() => dpsSkillBreakdownWindow?.skillRows[0]?.name ?? "--");
const updateIntervalSeconds = 0.2;

</script>

<svelte:window oncontextmenu={() => window.history.back()} />

{#if dpsSkillBreakdownWindow !== undefined}
  <section class="live-panel" aria-label="Damage skill breakdown">
    <div class="live-panel__header">
      <div class="live-panel__title">
        <span>Skill breakdown</span>
        {inspectedPlayer()?.name ?? "Unknown"}
      </div>
      <div class="live-panel__metrics">
        <span>Skills<strong>{skillCount()}</strong></span>
        <span>DPS<strong><AbbreviatedNumber num={inspectedPlayer()?.valuePerSec ?? 0} /></strong></span>
        <span>Damage<strong><AbbreviatedNumber num={inspectedPlayer()?.totalValue ?? 0} /></strong></span>
        <span>Top Skill<strong>{topSkillName()}</strong></span>
        <span>Refresh<strong>{updateIntervalSeconds}s</strong></span>
      </div>
    </div>

    {#each inspectedPlayerTable.getRowModel().rows as row (row.id)}
      {@const cells = row.getVisibleCells()}
      {@const playerInfoCell = cells.find((cell) => cell.column.id === "playerInfo")}
      {@const metricCells = cells.filter((cell) => cell.column.id !== "playerInfo")}
      <div class="live-panel__inspect" style={`--bar-color: ${inspectedClassColor()};`}>
        {#if playerInfoCell}
          <div class="live-panel__inspect-name">
            <FlexRender content={playerInfoCell.column.columnDef.cell ?? "UNKNOWN CELL"} context={playerInfoCell.getContext()} />
          </div>
        {/if}
        <div class="live-panel__inspect-metrics">
          {#each metricCells as cell (cell.id)}
            <span>
              <span>{cell.column.columnDef.meta?.label ?? cell.column.id}</span>
              <strong>
                <FlexRender content={cell.column.columnDef.cell ?? "UNKNOWN CELL"} context={cell.getContext()} />
              </strong>
            </span>
          {/each}
        </div>
      </div>
    {/each}

    <div class="live-panel__body">
      <div class="live-panel__scroll">
        <table class="live-table" aria-label="Skill breakdown">
          <thead>
            {#each dpsSkillBreakdownTable.getHeaderGroups() as headerGroup (headerGroup.id)}
              <tr>
                {#each headerGroup.headers as header (header.id)}
                  <th
                    scope="col"
                    class={`${header.column.columnDef.meta?.class ?? ""} ${header.column.id === "skillName" ? "player-head" : ""}`.trim()}
                  >
                    <FlexRender content={header.column.columnDef.header ?? "UNKNOWN HEADER"} context={header.getContext()} />
                  </th>
                {/each}
              </tr>
            {/each}
          </thead>
          <tbody>
            {#each dpsSkillBreakdownTable.getRowModel().rows as row (row.id)}
              {@const fillWidth = dpsSkillBreakdownWindow.topValue > 0
                ? Math.min(100, (row.original.totalValue / dpsSkillBreakdownWindow.topValue) * 100)
                : 0}
              <tr style={`--bar-color: ${inspectedClassColor()}; --fill: ${fillWidth}%;`}>
                {#each row.getVisibleCells() as cell (cell.id)}
                  <td class={`${cell.column.columnDef.meta?.class ?? ""} ${cell.column.id === "skillName" ? "player-cell" : ""}`.trim()}>
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
{/if}
