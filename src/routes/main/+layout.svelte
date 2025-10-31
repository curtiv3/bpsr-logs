<script lang="ts">
  import * as Sidebar from "$lib/components/ui/sidebar/index.js";

  import AppSidebar from "./sidebar.svelte";
  import Header from "./header.svelte";
  import { setupShortcuts } from "./settings/shortcuts";
  import { getCurrentWebviewWindow } from "@tauri-apps/api/webviewWindow";
  import { goto } from "$app/navigation";

  let { children } = $props();

  $effect.pre(() => {
    (async () => {
      await setupShortcuts();
    })();
  });

  const appWebview = getCurrentWebviewWindow();
  appWebview.listen<string>("navigate", (event) => {
    const route = event.payload;
    goto(route);
  });
</script>

<div class="text-foreground min-h-screen">
  <Sidebar.Provider>
    <AppSidebar />
    <Sidebar.Inset class="relative">
      <Header />
      <main class="px-4 py-10 lg:px-10">
        <div class="mx-auto flex w-full max-w-6xl flex-col gap-8">
          <section class="rounded-3xl border border-border/70 bg-card/80 p-6 shadow-xl backdrop-blur-xl lg:p-8">
            {@render children()}
          </section>
        </div>
      </main>
    </Sidebar.Inset>
  </Sidebar.Provider>
</div>
