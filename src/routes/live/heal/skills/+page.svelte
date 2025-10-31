<script lang="ts">
  import { onMount } from "svelte";
  import { commands, type SkillsWindow } from "$lib/bindings";
  import { getClassColor } from "$lib/utils.svelte";
  import { page } from "$app/state";
  import { createSvelteTable, FlexRender } from "$lib/svelte-table";
  import { healPlayersColumnDefs, healSkillsColumnDefs } from "$lib/table-info";
  import { getCoreRowModel } from "@tanstack/table-core";
  import { SETTINGS } from "$lib/settings-store";
  import AbbreviatedNumber from "$lib/components/abbreviated-number.svelte";

  const playerUid: string = page.url.searchParams.get("playerUid") ?? "-1";

  onMount(() => {
    fetchData();
    const interval = setInterval(fetchData, 200);
    return () => clearInterval(interval);
  });

  let healSkillBreakdownWindow: SkillsWindow = $state({ currPlayer: [], skillRows: [] });

  async function fetchData() {
    try {
      const result = SETTINGS.misc.state.testingMode ? await commands.getTestSkillWindow(playerUid) : await commands.getHealSkillWindow(playerUid);
      if (result.status !== "ok") {
        console.warn("Failed to get skill window: ", result.error);
        return;
      } else {
        healSkillBreakdownWindow = result.data;
        // console.log("healSkillBreakdown: ", +Date.now(), $state.snapshot(healSkillBreakdownWindow));
      }
    } catch (e) {
      console.error("Error fetching data: ", e);
    }
  }

  const currPlayerTable = createSvelteTable({
    get data() {
      return healSkillBreakdownWindow.currPlayer;
    },
    columns: healPlayersColumnDefs,
    getCoreRowModel: getCoreRowModel(),
    state: {
      get columnVisibility() {
        return SETTINGS.live.heal.skillBreakdown.state;
      },
    },
  });

const healSkillBreakdownTable = createSvelteTable({
  get data() {
    return healSkillBreakdownWindow.skillRows;
  },
  columns: healSkillsColumnDefs,
  getCoreRowModel: getCoreRowModel(),
  state: {
    get columnVisibility() {
      return SETTINGS.live.heal.skillBreakdown.state;
    },
  },
});

let maxSkillValue = $derived(healSkillBreakdownWindow.skillRows.reduce((max, p) => (p.totalDmg > max ? p.totalDmg : max), 0));

let SETTINGS_YOUR_NAME = $derived(SETTINGS.general.state.showYourName);
let SETTINGS_OTHERS_NAME = $derived(SETTINGS.general.state.showOthersName);

const skillCount = $derived(() => healSkillBreakdownWindow.skillRows.length);
const inspectedPlayer = $derived(() => healSkillBreakdownWindow.currPlayer[0]);
const inspectedClassColor = $derived(() => {
  const player = inspectedPlayer;
  if (!player) {
    return "transparent";
  }
  return getClassColor(player.className ?? "");
});
const topSkillName = $derived(() => healSkillBreakdownWindow.skillRows[0]?.name ?? "--");
const updateIntervalSeconds = 0.2;
</script>

<svelte:window oncontextmenu={() => window.history.back()} />

<section class="live-panel" aria-label="Healing skill breakdown">
  <div class="live-panel__header">
    <div class="live-panel__title">
      <span>Skill breakdown</span>
      {inspectedPlayer()?.name ?? "Unknown"}
    </div>
    <div class="live-panel__metrics">
      <span>Skills<strong>{skillCount()}</strong></span>
      <span>HPS<strong><AbbreviatedNumber num={inspectedPlayer()?.valuePerSec ?? 0} /></strong></span>
      <span>Heal<strong><AbbreviatedNumber num={inspectedPlayer()?.totalValue ?? 0} /></strong></span>
      <span>Top Skill<strong>{topSkillName()}</strong></span>
      <span>Refresh<strong>{updateIntervalSeconds}s</strong></span>
    </div>
  </div>

  {#each currPlayerTable.getRowModel().rows as row (row.id)}
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
      <table class="live-table" aria-label="Healing skill breakdown">
        <thead>
          {#each healSkillBreakdownTable.getHeaderGroups() as headerGroup (headerGroup.id)}
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
          {#each healSkillBreakdownTable.getRowModel().rows as row (row.id)}
            {@const currPlayer = healSkillBreakdownWindow.currPlayer[0]}
            {#if currPlayer}
              {@const isYou = row.original.name.includes("You")}
              {@const className = isYou
                ? (SETTINGS_YOUR_NAME !== "Hide Your Name" ? currPlayer.className : "Hidden Class")
                : SETTINGS_OTHERS_NAME !== "Hide Others' Name" ? currPlayer.className : "Hidden Class"}
              {@const classColor = getClassColor(className)}
              {@const fillWidth = SETTINGS.general.state.relativeToTopHealSkill
                ? (maxSkillValue > 0 ? Math.min(100, (row.original.totalDmg / maxSkillValue) * 100) : 0)
                : Math.min(100, row.original.dmgPct)}
              <tr style={`--bar-color: ${classColor}; --fill: ${fillWidth}%;`}>
                {#each row.getVisibleCells() as cell (cell.id)}
                  <td class={`${cell.column.columnDef.meta?.class ?? ""} ${cell.column.id === "skillName" ? "player-cell" : ""}`.trim()}>
                    <FlexRender content={cell.column.columnDef.cell ?? "UNKNOWN CELL"} context={cell.getContext()} />
                  </td>
                {/each}
              </tr>
            {/if}
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</section>
