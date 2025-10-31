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

<div class="rounded-3xl border border-border/70 bg-card/70 p-5 shadow-xs transition hover:border-border/60">
  <div class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
    <div class="space-y-1">
      <p class="text-sm font-medium text-foreground">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Button class="sm:ml-auto" {...restProps}>{buttonLabel}</Button>
  </div>
</div>
