<script lang="ts">
  import { Tabs as TabsPrimitive } from "bits-ui";
  import { cn } from "$lib/utils.js";

  let { ref = $bindable(null), class: className, ...restProps }: TabsPrimitive.TriggerProps = $props();
</script>

<TabsPrimitive.Trigger
  bind:ref
  data-slot="tabs-trigger"
  class={cn(
    "inline-flex h-8 flex-1 items-center justify-center gap-1.5 whitespace-nowrap rounded-lg px-3 text-sm font-medium text-muted-foreground transition-colors hover:text-foreground data-[state=active]:bg-foreground/10 data-[state=active]:text-foreground data-[state=active]:shadow-sm focus-visible:outline focus-visible:outline-1 focus-visible:outline-ring/60 focus-visible:ring-[3px] focus-visible:ring-ring/30 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0",
    className,
  )}
  {...restProps}
/>
