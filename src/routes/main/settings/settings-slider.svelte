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

<div class="rounded-3xl border border-border/70 bg-card/70 p-5 shadow-xs transition hover:border-border/60">
  <div class="flex flex-col gap-4">
    <div class="space-y-1">
      <p class="text-sm font-medium text-foreground">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <div class="flex flex-col-reverse gap-4 sm:flex-row sm:items-center sm:justify-between">
      <Slider type="single" bind:value min={0} max={120} step={5} {...restProps} class="sm:max-w-[70%]" />
      <div class="text-sm font-semibold text-foreground/70">
        <span>{value}s</span>
      </div>
    </div>
  </div>
</div>
