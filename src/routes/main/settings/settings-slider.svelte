<script lang="ts">
  import { Slider } from "$lib/components/ui/slider/index.js";

  let {
    label = "",
    description = "",
    value = $bindable(60),
    ...restProps
  }: {
    label: string;
    description?: string | undefined;
    value: number;
  } = $props();
</script>

<div class="rounded-2xl border border-border/60 bg-card/50 p-4 shadow-sm transition-colors hover:border-border/50">
  <div class="flex flex-col gap-3">
    <div class="space-y-1.5">
      <p class="text-sm font-semibold text-foreground/90">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <div class="flex flex-col-reverse gap-3 sm:flex-row sm:items-center sm:justify-between">
      <Slider type="single" bind:value min={0} max={120} step={5} {...restProps} class="sm:max-w-[65%]" />
      <div class="text-sm font-semibold text-foreground/80">
        <span>{value}s</span>
      </div>
    </div>
  </div>
</div>
