<script lang="ts">
  import Button, { type ButtonProps } from "$lib/components/ui/button/button.svelte";

  let {
    buttonLabel = "",
    label = "",
    description = "",
    ...restProps
  }: {
    buttonLabel: string;
    label: string;
    description?: string | undefined;
  } & ButtonProps = $props();
</script>

<div class="rounded-2xl border border-border/60 bg-card/50 p-4 shadow-sm transition-colors hover:border-border/50">
  <div class="flex flex-col gap-3 sm:grid sm:grid-cols-[minmax(0,1fr)_auto] sm:items-center sm:gap-4">
    <div class="space-y-1.5">
      <p class="text-sm font-semibold text-foreground/90">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Button class="sm:ml-auto" {...restProps}>{buttonLabel}</Button>
  </div>
</div>
