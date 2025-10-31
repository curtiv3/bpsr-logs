<script lang="ts">
  import * as Sidebar from "$lib/components/ui/sidebar/index.js";

  import { getVersion } from "@tauri-apps/api/app";
  import { SIDEBAR_ROUTES } from "./routes.svelte";
</script>

<Sidebar.Root class="shadow-xl">
  <Sidebar.Header class="px-6 pt-8">
    <span class="text-xs font-semibold uppercase tracking-[0.3em] text-muted-foreground">BPSR</span>
    <h2 class="text-2xl font-semibold tracking-tight text-sidebar-foreground">Logs</h2>
  </Sidebar.Header>
  <Sidebar.Separator class="mx-6 border-t border-sidebar-border/70" />
  <Sidebar.Content class="px-4">
    <Sidebar.Group>
      <Sidebar.GroupContent>
        <Sidebar.Menu class="mt-2 space-y-1">
          {#each Object.entries(SIDEBAR_ROUTES) as [href, route] (route.label)}
            <Sidebar.MenuItem>
              <Sidebar.MenuButton class="rounded-2xl py-2 text-sm font-medium transition-all hover:shadow-sm data-[active=true]:shadow-sm">
                {#snippet child({ props })}
                  <a {href} {...props} class="flex items-center gap-3 rounded-2xl px-3 py-2">
                    <route.icon class="size-4" />
                    <span>{route.label}</span>
                  </a>
                {/snippet}
              </Sidebar.MenuButton>
            </Sidebar.MenuItem>
          {/each}
        </Sidebar.Menu>
      </Sidebar.GroupContent>
    </Sidebar.Group>
  </Sidebar.Content>
  <Sidebar.Footer class="px-6 pb-8 text-xs font-medium text-sidebar-foreground/60">
    <span>v{#await getVersion()}X.Y.Z{:then version}{version}{/await}</span>
  </Sidebar.Footer>
</Sidebar.Root>
