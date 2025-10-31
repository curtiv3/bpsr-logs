<script lang="ts">
  import CheckIcon from "@lucide/svelte/icons/check";
  import ChevronsUpDownIcon from "@lucide/svelte/icons/chevrons-up-down";
  import * as Command from "$lib/components/ui/command/index.js";
  import * as Popover from "$lib/components/ui/popover/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import { cn } from "$lib/utils.js";

  let {
    label = "",
    description = "",
    selected = $bindable(""),
    values = [],
  }: {
    label: string;
    description?: string | undefined;
    selected: string;
    values: string[];
  } = $props();

  let open = $state(false);
</script>

<div class="rounded-3xl border border-border/70 bg-card/70 p-5 shadow-xs transition hover:border-border/60">
  <!-- https://shadcn-svelte.com/docs/components/combobox -->
  <div class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
    <div class="space-y-1">
      <p class="text-sm font-medium text-foreground">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Popover.Root bind:open>
      <Popover.Trigger class="sm:ml-auto">
        <Button variant="outline" class="w-full justify-between sm:w-[220px]" role="combobox">
          {selected}
          <ChevronsUpDownIcon />
        </Button>
      </Popover.Trigger>
      <Popover.Content class="w-[220px] p-0">
        <Command.Root>
          <Command.List>
            <Command.Group>
              {#each values as value (value)}
                <Command.Item
                  {value}
                  onSelect={() => {
                    selected = value;
                    open = false;
                  }}
                >
                  <CheckIcon class={cn(selected !== value && "text-transparent")} />
                  {value}
                </Command.Item>
              {/each}
            </Command.Group>
          </Command.List>
        </Command.Root>
      </Popover.Content>
    </Popover.Root>
  </div>
</div>
