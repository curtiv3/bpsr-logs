<script lang="ts">
  import { Tabs as TabsPrimitive } from "bits-ui";
  import { cn } from "$lib/utils.js";

  let { ref = $bindable(null), class: className, ...restProps }: TabsPrimitive.TriggerProps = $props();
</script>

<TabsPrimitive.Trigger
  bind:ref
  data-slot="tabs-trigger"
  class={cn(
    "data-[state=active]:bg-foreground/5 data-[state=active]:text-foreground focus-visible:border-ring focus-visible:ring-ring/40 focus-visible:outline-ring inline-flex h-9 flex-1 items-center justify-center gap-1.5 whitespace-nowrap rounded-full px-4 text-sm font-medium text-muted-foreground transition-all hover:text-foreground focus-visible:outline-1 focus-visible:ring-[3px] data-[state=active]:shadow-sm [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0",
    className,
  )}
  {...restProps}
/>
