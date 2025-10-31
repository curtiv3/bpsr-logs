<script lang="ts">
  import { getCurrentWebviewWindow, WebviewWindow } from "@tauri-apps/api/webviewWindow";

  import CameraIcon from "virtual:icons/lucide/camera";
  import TimerResetIcon from "virtual:icons/lucide/timer-reset";
  import PauseIcon from "virtual:icons/lucide/pause";
  import PlayIcon from "virtual:icons/lucide/play";
  import MinusIcon from "virtual:icons/lucide/minus";
  import PointerIcon from "virtual:icons/lucide/pointer";
  import SettingsIcon from "virtual:icons/lucide/settings";
  import RefreshCwIcon from "virtual:icons/lucide/refresh-cw";

  import { onMount, tick } from "svelte";
  import { commands, type HeaderInfo } from "$lib/bindings";
  import { takeScreenshot, tooltip } from "$lib/utils.svelte";
  import AbbreviatedNumber from "$lib/components/abbreviated-number.svelte";
  import { emitTo } from "@tauri-apps/api/event";
  import { SETTINGS } from "$lib/settings-store";

  onMount(() => {
    fetchData();
    const interval = setInterval(fetchData, 200);
    return () => clearInterval(interval);
  });

  let hasReset = false;

  async function fetchData() {
    // TODO: there's a bug here where if headerInfo.timeLastCombatPacketMs is 0 at initial load it just resets instantly
    if (SETTINGS.general.state.resetElapsed && !hasReset && Date.now() - headerInfo.timeLastCombatPacketMs > SETTINGS.general.state.resetElapsed * 1000) {
      hasReset = true;
      console.log(`Resetting as ${SETTINGS.general.state.resetElapsed}s has passed.`);
      commands.hardReset(); // TODO: this is temporary, switch to resetEncounter once bug is fixed.
    }
    try {
      const result = await commands.getHeaderInfo();
      if (result.status !== "ok") {
        console.warn("Failed to get header: ", result.error);
        return;
      } else {
        headerInfo = result.data;
        // console.log("header: ", +Date.now(), $state.snapshot(headerInfo));
        if (hasReset) {
          hasReset = false;
          window.location.reload();
          console.log("Fresh packet");
        }
      }
    } catch (e) {
      console.error("Error fetching data: ", e);
    }
  }

  function formatElapsed(msElapsed: number) {
    const totalSeconds = Math.floor(Number(msElapsed) / 1000);
    const minutes = Math.floor((totalSeconds % 3600) / 60);
    const seconds = totalSeconds % 60;

    return `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}`;
  }

  let headerInfo: HeaderInfo = $state({
    totalDps: 0,
    totalDmg: 0,
    elapsedMs: 0,
    timeLastCombatPacketMs: Date.now(), // TODO: tempfix
  });
  let isEncounterPaused = $state(false);
  const {
    screenshotDiv,
  }: {
    screenshotDiv?: HTMLElement;
  } = $props();
  const appWindow = getCurrentWebviewWindow();

  async function openSettings() {
    const mainWindow = await WebviewWindow.getByLabel("main");
    if (mainWindow !== null) {
      await mainWindow?.unminimize();
      await mainWindow?.show();
      await mainWindow?.setFocus();
      await emitTo("main", "navigate", "/main/settings"); // main/+layout.svelte
    }
  }
</script>

<!-- justify-between to create left/right sides -->
<header data-tauri-drag-region class="live-app-header">
  <div class="live-app-header__metrics">
    <button
      type="button"
      class="live-icon-button"
      onclick={() => {
        commands.hardReset();
        window.location.reload();
      }}
      aria-label="Hard reset encounter"
      {@attach tooltip(() => "Temp Fix: Hard Reset")}
    >
      <RefreshCwIcon />
    </button>
    <div>
      <span {@attach tooltip(() => "Time Elapsed")}>Elapsed</span>
      <strong>{formatElapsed(headerInfo.elapsedMs)}</strong>
    </div>
    <div>
      <span {@attach tooltip(() => "Total Damage Dealt")}>T.DMG</span>
      <strong {@attach tooltip(() => headerInfo.totalDmg.toLocaleString())}>
        <AbbreviatedNumber num={Number(headerInfo.totalDmg)} />
      </strong>
    </div>
    <div>
      <span {@attach tooltip(() => "Total Damage per Second")}>T.DPS</span>
      <strong {@attach tooltip(() => headerInfo.totalDps.toLocaleString())}>
        <AbbreviatedNumber num={headerInfo.totalDps} />
      </strong>
    </div>
  </div>

  <div class="live-app-header__controls">
    <button
      type="button"
      class="live-icon-button"
      onclick={async () => {
        const prev = SETTINGS.general.state.showOthersName;
        if (SETTINGS.general.state.showOthersName === "Show Others' Name") {
          SETTINGS.general.state.showOthersName = "Show Others' Class";
        }

        // Wait for reactive flush & paint
        await tick();

        // Take screenshot AFTER change is visible
        await takeScreenshot(screenshotDiv);

        // Revert & let UI update
        SETTINGS.general.state.showOthersName = prev;
        await tick();
      }}
      aria-label="Screenshot to clipboard"
      {@attach tooltip(() => "Screenshot to Clipboard")}
    >
      <CameraIcon />
    </button>
    <button
      type="button"
      class="live-icon-button"
      onclick={async () => {
        commands.resetEncounter();
        window.location.reload(); // TODO: temp fix
      }}
      aria-label="Reset encounter"
      {@attach tooltip(() => "Reset Encounter")}
    >
      <TimerResetIcon />
    </button>
    <button
      type="button"
      class="live-icon-button"
      onclick={() => {
        commands.togglePauseEncounter();
        isEncounterPaused = !isEncounterPaused;
      }}
      aria-label={isEncounterPaused ? "Resume encounter" : "Pause encounter"}
    >
      {#if isEncounterPaused}
        <PlayIcon {@attach tooltip(() => "Resume Encounter")} />
      {:else}
        <PauseIcon {@attach tooltip(() => "Pause Encounter")} />
      {/if}
    </button>
    <button
      type="button"
      class="live-icon-button"
      onclick={() => appWindow.setIgnoreCursorEvents(true)}
      aria-label="Enable clickthrough"
      {@attach tooltip(() => "Clickthrough")}
    >
      <PointerIcon />
    </button>
    <button
      type="button"
      class="live-icon-button"
      onclick={() => openSettings()}
      aria-label="Open settings"
      {@attach tooltip(() => "Settings")}
    >
      <SettingsIcon />
    </button>
    <button
      type="button"
      class="live-icon-button"
      onclick={() => appWindow.hide()}
      aria-label="Minimize overlay"
      {@attach tooltip(() => "Minimize")}
    >
      <MinusIcon />
    </button>
  </div>
</header>
