<script lang="ts">
  import { Switch } from "$lib/components/ui/switch/index.js";

  let {
    checked = $bindable(false),
    label = "",
  }: {
    checked: boolean | undefined;
    label: string;
  } = $props();
</script>

<div class="rounded-3xl border border-border/70 bg-card/70 p-5 shadow-xs transition hover:border-border/60">
  <div class="flex items-center justify-between gap-4">
    <p class="text-sm font-medium text-foreground">{label}</p>
    <Switch bind:checked class="shrink-0" />
  </div>
</div>
