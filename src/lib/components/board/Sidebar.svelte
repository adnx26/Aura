<script>
  import { visibleNodes } from '$lib/stores/filters.js';
  import { activeId } from '$lib/stores/activeId.js';
  import { openPanel } from '$lib/stores/panel.js';
  import { panTarget } from '$lib/stores/board.js';
  import { AGENTS, SUB_AGENTS, DEVELOPERS } from '$lib/data.js';

  let visible = typeof localStorage !== 'undefined'
    ? localStorage.getItem('aura-sidebar-visible') !== 'false'
    : true;

  $: if (typeof localStorage !== 'undefined') localStorage.setItem('aura-sidebar-visible', visible);

  let openKeys = new Set(
    typeof localStorage !== 'undefined'
      ? JSON.parse(localStorage.getItem('aura-sidebar-open') || '[]')
      : []
  );

  function toggleKey(key) {
    if (openKeys.has(key)) openKeys.delete(key);
    else openKeys.add(key);
    openKeys = openKeys;
    if (typeof localStorage !== 'undefined')
      localStorage.setItem('aura-sidebar-open', JSON.stringify([...openKeys]));
  }

  function onRowClick(id) {
    activeId.set(id);
    openPanel({ mode:'agent', id });
    panTarget.set(id);
  }

  $: tree = $visibleNodes.tasks.map(task => {
    const dev   = DEVELOPERS.find(d => d.id === task.developerId);
    const agent = AGENTS.find(a => a.id === task.agentId);
    const subs  = agent ? SUB_AGENTS.filter(sa => sa.parentId === agent.id) : [];
    const taskKey  = task.id;
    const devKey   = dev   ? `${task.id}|${dev.id}`   : null;
    const agentKey = agent ? `${task.id}|${agent.id}` : null;
    return { task, dev, agent, subs, taskKey, devKey, agentKey };
  });
</script>

<div
  class="relative flex-shrink-0 bg-aura-surface border-r border-white/[0.065] flex flex-col transition-[width] duration-[280ms] z-10"
  style="width:{visible?'clamp(228px,22vw,268px)':'0'};overflow:visible;background:linear-gradient(180deg,rgba(13,22,38,0.96),rgba(11,18,31,0.93) 40%,rgba(9,15,28,0.96));box-shadow:inset -1px 0 0 rgba(148,163,184,0.10), 16px 0 30px rgba(4,8,18,0.16)"
>
  <!-- Tree content -->
  <div class="px-4 pt-5 pb-4 border-b border-slate-600/20">
    <div class="rounded-[18px] border border-slate-600/18 bg-slate-900/38 px-4 py-4 text-center shadow-[inset_0_1px_0_rgba(255,255,255,0.03)]">
      <div class="font-syne text-[1.04rem] tracking-[.02em] text-slate-100">Structure View</div>
      <div class="text-[.68rem] leading-[1.5] text-slate-400 mt-2">Browse tasks, owners, and agents in one clean hierarchy.</div>
    </div>
  </div>

  <div class="flex-1 py-4 overflow-y-auto overflow-x-hidden" style="scrollbar-width:thin">
    {#each tree as { task, dev, agent, subs, taskKey, devKey, agentKey }}
      <!-- Task row -->
      <div
        class="flex items-center gap-[8px] px-3 pr-5 py-[10px] mx-3 rounded-[14px] cursor-pointer text-[.78rem] leading-[1.4] whitespace-nowrap overflow-hidden transition-colors {$activeId === task.id ? 'bg-slate-900/90 border border-amber-400/20 shadow-[0_10px_22px_rgba(4,8,18,0.26)]' : 'border border-transparent hover:bg-slate-800/55'}"
        role="button" tabindex="0"
        on:click={() => onRowClick(task.id)}
        on:keydown={e => e.key==='Enter' && onRowClick(task.id)}
      >
        <span
          class="flex-shrink-0 w-6 h-6 flex items-center justify-center text-[.72rem] text-slate-300 cursor-pointer hover:text-white transition-colors rounded-full bg-slate-800/70 border border-slate-600/30 hover:bg-slate-700/60 {!dev ? 'opacity-0 pointer-events-none' : ''}"
          on:click|stopPropagation={() => dev && toggleKey(taskKey)}
          role="button" tabindex="-1"
          on:keydown={e => e.key==='Enter' && dev && toggleKey(taskKey)}
        >{dev ? (openKeys.has(taskKey) ? '▾' : '▸') : '▸'}</span>
        <span class="overflow-hidden text-ellipsis flex-1 font-medium text-slate-100">{task.title}</span>
        {#if dev}
          <span class="text-[.56rem] uppercase tracking-[.16em] text-slate-600/70">Expand</span>
        {/if}
      </div>

      <!-- Dev children -->
      {#if dev && openKeys.has(taskKey)}
        <div>
          <div
            class="flex items-center gap-[8px] mx-3 rounded-[14px] cursor-pointer text-[.76rem] leading-[1.4] whitespace-nowrap overflow-hidden transition-colors {$activeId === dev.id ? 'bg-slate-900/90 border border-sky-400/20 shadow-[0_10px_22px_rgba(4,8,18,0.26)]' : 'border border-transparent hover:bg-slate-800/55'}"
            style="padding:8px 12px 8px {12+18}px"
            role="button" tabindex="0"
            on:click={() => onRowClick(dev.id)}
            on:keydown={e => e.key==='Enter' && onRowClick(dev.id)}
          >
            <span
              class="flex-shrink-0 w-6 h-6 flex items-center justify-center text-[.72rem] text-slate-300 cursor-pointer hover:text-white transition-colors rounded-full bg-slate-800/70 border border-slate-600/30 hover:bg-slate-700/60 {!agent ? 'opacity-0 pointer-events-none' : ''}"
              on:click|stopPropagation={() => agent && toggleKey(devKey)}
              role="button" tabindex="-1"
              on:keydown={e => e.key==='Enter' && agent && toggleKey(devKey)}
            >{agent ? (openKeys.has(devKey) ? '▾' : '▸') : '▸'}</span>
            <span class="overflow-hidden text-ellipsis flex-1 font-medium text-slate-200">{dev.name}</span>
          </div>

          <!-- Agent children -->
          {#if agent && openKeys.has(devKey)}
            <div>
              <div
                class="flex items-center gap-[8px] mx-3 rounded-[14px] cursor-pointer text-[.74rem] leading-[1.4] whitespace-nowrap overflow-hidden transition-colors {$activeId === agent.id ? 'bg-slate-900/90 border border-violet-400/20 shadow-[0_10px_22px_rgba(4,8,18,0.26)]' : 'border border-transparent hover:bg-slate-800/55'}"
                style="padding:8px 12px 8px {12+36}px"
                role="button" tabindex="0"
                on:click={() => onRowClick(agent.id)}
                on:keydown={e => e.key==='Enter' && onRowClick(agent.id)}
              >
                <span
                  class="flex-shrink-0 w-6 h-6 flex items-center justify-center text-[.72rem] text-slate-300 cursor-pointer hover:text-white transition-colors rounded-full bg-slate-800/70 border border-slate-600/30 hover:bg-slate-700/60 {!subs.length ? 'opacity-0 pointer-events-none' : ''}"
                  on:click|stopPropagation={() => subs.length && toggleKey(agentKey)}
                  role="button" tabindex="-1"
                  on:keydown={e => e.key==='Enter' && subs.length && toggleKey(agentKey)}
                >{subs.length ? (openKeys.has(agentKey) ? '▾' : '▸') : '▸'}</span>
                <span class="overflow-hidden text-ellipsis flex-1 font-medium text-slate-200">{agent.name}</span>
              </div>

              <!-- Sub-agent children -->
              {#if subs.length && openKeys.has(agentKey)}
                {#each subs as sa}
                  <div
                    class="flex items-center gap-[8px] mx-3 rounded-[14px] cursor-pointer text-[.72rem] leading-[1.4] whitespace-nowrap overflow-hidden transition-colors {$activeId === sa.id ? 'bg-slate-900/90 border border-rose-400/20 shadow-[0_10px_22px_rgba(4,8,18,0.26)]' : 'border border-transparent hover:bg-slate-800/55'}"
                    style="padding:8px 12px 8px {12+54}px"
                    role="button" tabindex="0"
                    on:click={() => onRowClick(sa.id)}
                    on:keydown={e => e.key==='Enter' && onRowClick(sa.id)}
                  >
                    <span class="flex-shrink-0 w-6 opacity-0">▸</span>
                    <span class="overflow-hidden text-ellipsis flex-1 font-medium text-slate-300">{sa.name}</span>
                  </div>
                {/each}
              {/if}
            </div>
          {/if}
        </div>
      {/if}
    {/each}
  </div>

  <!-- Collapse toggle -->
  <button
    class="absolute -right-[22px] top-1/2 -translate-y-1/2 w-[22px] h-14 bg-slate-900/95 border border-slate-600/30 border-l-0 rounded-r-[12px] flex items-center justify-center cursor-pointer text-slate-400 text-[.9rem] z-[11] hover:text-slate-100 hover:bg-slate-800 select-none transition-colors shadow-[10px_0_22px_rgba(4,8,18,0.22)]"
    on:click={() => visible = !visible}
  >{visible ? '‹' : '›'}</button>
</div>
