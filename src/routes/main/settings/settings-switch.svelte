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

<div class="rounded-3xl border border-border/70 bg-card/70 p-5 shadow-xs transition hover:border-border/60">
  <div class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
    <div class="space-y-1">
      <p class="text-sm font-medium text-foreground">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Switch bind:checked class="sm:ml-auto" {...restProps} />
  </div>
</div>
