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

<div class="rounded-2xl border border-border/60 bg-card/50 p-4 shadow-sm transition-colors hover:border-border/50">
  <div class="flex items-center justify-between gap-3">
    <p class="text-sm font-semibold text-foreground/90">{label}</p>
    <Switch bind:checked class="shrink-0" />
  </div>
</div>
