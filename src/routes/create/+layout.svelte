<script>
	import { AppShell } from '@skeletonlabs/skeleton';
</script>

<!-- App Shell -->
<AppShell slotSidebarLeft="bg-surface-500/5 w-56 p-4">
	<!-- Page Route Content -->
    <svelte:fragment slot="sidebarRight">
        <slot name="sidebarRight"></slot>
    </svelte:fragment>
	<slot />
</AppShell>