<script lang="ts">
	import "../app.pcss";
	import { ModeWatcher } from "mode-watcher";

	import Sun from "svelte-radix/Sun.svelte";
  import Moon from "svelte-radix/Moon.svelte";
 
  import { resetMode, setMode } from "mode-watcher";
  import { Button } from "$lib/components/ui/button/index.js";
  import * as DropdownMenu from "$lib/components/ui/dropdown-menu/index.js";
</script>

<ModeWatcher defaultMode="dark" />
<div class="flex h-full flex-col">

	<!-- Mode switcher toggle group -->
	<div class="absolute top-0 right-0 h-16 w-16 p-2 m-2">
		<DropdownMenu.Root>
			<DropdownMenu.Trigger asChild let:builder>
			<Button builders={[builder]} variant="outline" size="icon">
				<Sun
				class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
				/>
				<Moon
				class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
				/>
				<span class="sr-only">Toggle theme</span>
			</Button>
			</DropdownMenu.Trigger>
			<DropdownMenu.Content align="end">
			<DropdownMenu.Item on:click={() => setMode("light")}
				>Light</DropdownMenu.Item
			>
			<DropdownMenu.Item on:click={() => setMode("dark")}>Dark</DropdownMenu.Item>
			<DropdownMenu.Item on:click={() => resetMode()}>System</DropdownMenu.Item>
			</DropdownMenu.Content>
		</DropdownMenu.Root>
	</div>

	<!-- Main Content -->
	<div class="w-full flex-grow px-2 sm:px-4">
		<div class="container mx-auto">
			<slot />
		</div>
	</div>

	<!-- Footer -->
	<footer class="py-6 md:px-8 md:py-0">
		<div class="container flex flex-col items-center justify-center gap-4 md:h-24 md:flex-row">
			<div class="flex flex-col items-center gap-4 px-8 text-center md:gap-2 md:px-0">
				<p class="text-center text-sm leading-loose text-muted-foreground">
					If you like this website, check out the repository on 
					<a href="https://github.com/huntabyte/shadcn-svelte" target="_blank" rel="noreferrer" class="font-medium underline underline-offset-4">GitHub</a>
					and be sure to leave a star! See the 
					<a href="/licensing" rel="noreferrer" class="font-medium underline underline-offset-4">Licensing</a>
					 page for the full list of attributions.
				</p>
			</div>
		</div>
	</footer>
	

</div>
