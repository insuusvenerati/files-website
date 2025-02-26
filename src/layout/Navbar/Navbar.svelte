<script lang="ts">
	import { navigating, page } from "$app/stores";
	import type { NavbarItem } from "$data/links";
	import { _ } from "svelte-i18n";
	import {
		defaultI18nValues,
		externalLink,
		isUrlContainPath,
		TreeView,
	} from "$lib";
	import { ListItem, Tooltip } from "fluent-svelte";
	import Navigation from "~icons/fluent/navigation-24-regular";
	import type { SvelteComponent } from "svelte";
	import type { SvelteHTMLElements } from "svelte/elements";

	export let items: NavbarItem[] = [];
	export let buttons: {
		icon: typeof SvelteComponent<SvelteHTMLElements["svg"]>;
		href: string;
		label: string;
	}[] = [];

	let innerWidth = 649; // Don't render the mobile layout before hydration
	let sidebarVisible = false;
	let sidebar: HTMLElement;
	let sidebarButton: HTMLButtonElement;

	const toggleSidebar = () => {
		sidebarVisible = !sidebarVisible;
	};

	const handleOuterClick = (e: MouseEvent) => {
		if (
			!(
				!sidebarVisible ||
				e.target === sidebarButton ||
				sidebarButton.contains(e.target as Node) ||
				e.target === sidebar ||
				sidebar.contains(e.target as Node)
			)
		) {
			toggleSidebar();
		}
	};

	$: {
		$navigating && (sidebarVisible = false);
	}
</script>

<svelte:window bind:innerWidth on:click={handleOuterClick} />

<header class="navbar">
	<nav class="inner">
		<a class="logo" href="/">
			<picture>
				<source
					media="(prefers-color-scheme: dark)"
					srcset="/branding/logo-dark.svg"
				/>
				<source
					media="(prefers-color-scheme: light)"
					srcset="/branding/logo-light.svg"
				/>
				<img
					alt="Files logo"
					class="logo-image"
					height="32"
					src="/branding/logo-light.svg"
					width="32"
				/>
			</picture>
			Files
		</a>
		{#if innerWidth > 648}
			<div class="divider" />
			{#each items as { name, path, external, icon, type }}
				{#if type === "divider"}
					<div class="divider" />
				{:else}
					<a
						class="item"
						class:selected={isUrlContainPath($page.url.pathname, path)}
						href={path}
						target={external ? "_blank" : undefined}
						rel={external ? "noreferrer noopener" : undefined}
					>
						{#if icon}
							<svelte:component this={icon} />
						{/if}
						<span>{name}</span>
					</a>
				{/if}
			{/each}
		{/if}
	</nav>
	<div class="buttons">
		<Tooltip
			text={$_("navbar.ukraine", defaultI18nValues)}
			placement="bottom"
			delay={200}
			offset={0}
		>
			<span
				class="support-ukraine"
				aria-label="Ukraine flag in support of Ukraine and its people, #supportukraine"
			/>
		</Tooltip>
		{#if innerWidth > 648}
			{#each buttons as { icon, href, label }}
				<a
					class="button"
					{href}
					aria-label={label}
					title={label}
					{...externalLink}
				>
					<svelte:component this={icon} />
				</a>
			{/each}
		{:else}
			<button
				on:click={toggleSidebar}
				bind:this={sidebarButton}
				class="button sidebar-button"
			>
				<Navigation />
			</button>
		{/if}
	</div>
	<aside
		bind:this={sidebar}
		class="sidebar scroller"
		class:visible={sidebarVisible}
	>
		{#each items as { name, path, external, sidebarTree, icon, type }}
			{#if type === "divider"}
				<hr />
			{:else if !sidebarTree}
				<ListItem
					type="navigation"
					on:click={toggleSidebar}
					selected={isUrlContainPath($page.url.pathname, path)}
					href={path}
					target={external ? "_blank" : undefined}
					rel={external ? "noreferrer noopener" : undefined}
				>
					<svelte:fragment slot="icon">
						{#if icon}
							<svelte:component this={icon} />
						{/if}
					</svelte:fragment>
					<span>{name}</span>
				</ListItem>
			{:else}
				<TreeView
					on:click={toggleSidebar}
					tree={[
						{
							title: name,
							icon,
							path: "",
							pages: [...sidebarTree],
						},
					]}
				/>
			{/if}
		{/each}
		<hr />
		{#each buttons as { icon, href, label }}
			<ListItem {href} type="navigation" {...externalLink}>
				<svelte:fragment slot="icon">
					{#if icon}
						<svelte:component this={icon} />
					{/if}
				</svelte:fragment>
				<span>{label}</span>
			</ListItem>
		{/each}
	</aside>
</header>

<style lang="scss">
	@use "./Navbar";
</style>
