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

<div class="rounded-2xl border border-border/60 bg-card/50 p-4 shadow-sm transition-colors hover:border-border/50">
  <!-- https://shadcn-svelte.com/docs/components/combobox -->
  <div class="flex flex-col gap-3 sm:grid sm:grid-cols-[minmax(0,1fr)_auto] sm:items-center sm:gap-4">
    <div class="space-y-1.5">
      <p class="text-sm font-semibold text-foreground/90">{label}</p>
      {#if description}
        <p class="text-muted-foreground text-xs leading-relaxed">{description}</p>
      {/if}
    </div>
    <Popover.Root bind:open>
      <Popover.Trigger class="sm:ml-auto">
        <Button variant="outline" class="w-full justify-between sm:min-w-[200px] sm:w-auto" role="combobox">
          <span class="truncate text-sm font-medium">{selected}</span>
          <ChevronsUpDownIcon class="shrink-0" />
        </Button>
      </Popover.Trigger>
      <Popover.Content class="min-w-[200px] p-0">
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
