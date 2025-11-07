<script lang="ts">
  import { Switch } from "$lib/components/ui/switch/index.js";
  import type { WithoutChildrenOrChild } from "$lib/utils";
  import { Switch as SwitchPrimitive } from "bits-ui";

  export type SettingsSwitchProps = {
    label: string;
    description?: string | undefined;
    checked: boolean;
  } & WithoutChildrenOrChild<SwitchPrimitive.RootProps>;

  let {
    label = "",
    description = "",
    checked = $bindable(false),
    ...restProps
  }: SettingsSwitchProps = $props();
</script>

<div class="rounded-2xl border border-border/60 bg-card/50 p-4 shadow-sm transition-colors hover:border-border/50">
  <div class="flex flex-col gap-3 sm:grid sm:grid-cols-[minmax(0,1fr)_auto] sm:items-center sm:gap-4">
    <div class="space-y-1.5">
      <p class="text-sm font-semibold text-foreground/90">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Switch bind:checked class="sm:ml-auto" {...restProps} />
  </div>
</div>
