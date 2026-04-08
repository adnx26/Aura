<script>
  import '../app.css';
  import Header from '$lib/components/Header.svelte';
  import DetailPanel from '$lib/components/DetailPanel.svelte';
</script>

<div class="shell flex flex-col h-screen overflow-hidden">
  <Header />
  <div class="app-content flex-1 min-h-0">
    <slot />
  </div>
  <DetailPanel />
</div>
