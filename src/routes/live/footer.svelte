<script lang="ts">
  import { page } from "$app/state";
  import { goto } from "$app/navigation";
  import { resolve } from "$app/paths";
  import { getVersion } from "@tauri-apps/api/app";
</script>

<footer class="live-app-footer">
  <div class="live-app-footer__tabs">
    <button
      type="button"
      class={`live-app-footer__tab ${page.url.pathname.includes("dps") ? "live-app-footer__tab--active" : ""}`}
      onclick={() => {
        goto(resolve("/live/dps"));
      }}>DPS</button
    >
    <button
      type="button"
      class={`live-app-footer__tab ${page.url.pathname.includes("heal") ? "live-app-footer__tab--active" : ""}`}
      onclick={() => {
        goto(resolve("/live/heal"));
      }}>HEAL</button
    >
  </div>
  <span class="live-app-footer__version">BPSR Logs v{#await getVersion()}X.Y.Z{:then version}{version}{/await}</span>
</footer>
