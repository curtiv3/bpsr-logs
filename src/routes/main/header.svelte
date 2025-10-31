<script lang="ts">
  import { Separator } from "$lib/components/ui/separator/index.js";
  import * as Sidebar from "$lib/components/ui/sidebar/index.js";

  import { page } from "$app/state";

  import { SIDEBAR_ROUTES } from "./routes.svelte";

  let currentPage = $derived(SIDEBAR_ROUTES[page.url.pathname]);
</script>

<header
  class="h-(--header-height) group-has-data-[collapsible=icon]/sidebar-wrapper:h-(--header-height) sticky top-0 z-30 flex shrink-0 items-center border-b border-border/60 bg-white/70 backdrop-blur-xl transition-[width,height] ease-linear"
>
  <div class="flex w-full items-center gap-3 px-5 py-3 lg:px-8">
    <Sidebar.Trigger class="-ml-1 rounded-full border border-transparent bg-white/70 p-2 shadow-xs backdrop-blur hover:border-border/80" />
    <Separator orientation="vertical" class="hidden h-6 w-px lg:block" />
    <div class="flex flex-col">
      <span class="text-xs font-medium uppercase tracking-[0.32em] text-muted-foreground">Dashboard</span>
      <h1 class="text-lg font-semibold tracking-tight text-foreground lg:text-xl">{currentPage?.label ?? "UNKNOWN PAGE LABEL"}</h1>
    </div>
  </div>
</header>
