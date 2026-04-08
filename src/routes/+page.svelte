<script>
  import BoardCanvas from '$lib/components/board/BoardCanvas.svelte';
  import Sidebar from '$lib/components/board/Sidebar.svelte';

  let boardCanvas;

  const legendItems = [
    { color:'#f59e0b', label:'Tasks' },
    { color:'#1dd4ef', label:'Developers' },
    { color:'#a78bfa', label:'Agents' },
    { color:'#fb7185', label:'Sub-Agents' }
  ];
</script>

<div class="application-frame flex flex-1 overflow-hidden relative z-[1]">
  <Sidebar />
  <BoardCanvas bind:this={boardCanvas} />
</div>

<!-- Zoom controls -->
<div class="fixed bottom-5 right-5 z-[25] flex flex-col gap-1">
  {#each [
    { label:'+',  title:'Zoom in',       action: () => boardCanvas?.zoomIn()  },
    { label:'⊡', title:'Fit to screen', action: () => boardCanvas?.zoomFit() },
    { label:'−',  title:'Zoom out',      action: () => boardCanvas?.zoomOut() },
  ] as btn}
    <button
      title={btn.title}
      on:click={btn.action}
      class="w-9 h-9 bg-slate-950/70 border border-slate-700/60 rounded-[10px] text-slate-300 cursor-pointer flex items-center justify-center text-base hover:text-white hover:border-slate-500/70 hover:bg-slate-900/85 transition-colors shadow-[0_12px_24px_rgba(4,8,18,0.22)] backdrop-blur-[10px]"
    >{btn.label}</button>
  {/each}
</div>

<div class="fixed bottom-5 left-1/2 -translate-x-1/2 z-[24] flex items-center gap-5 px-5 py-3 rounded-full border border-slate-700/60 bg-[rgba(7,13,24,0.82)] backdrop-blur-[16px] shadow-[0_18px_40px_rgba(4,8,18,0.28)]">
  {#each legendItems as item}
    <span class="flex items-center gap-2 text-[.72rem] uppercase tracking-[.12em] text-slate-300 whitespace-nowrap">
      <span class="inline-block w-[8px] h-[8px] rounded-full" style="background:{item.color};box-shadow:0 0 0 4px {item.color}22"></span>
      {item.label}
    </span>
  {/each}
</div>
