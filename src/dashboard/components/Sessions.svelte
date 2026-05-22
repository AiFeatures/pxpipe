<script lang="ts">
  // Compact horizontal bar chart of the conversations the proxy helped most.
  // Top-N sessions by estimated input tokens saved. Read-only — no navigation,
  // no detail page; the bars ARE the view. Data comes from the same
  // /api/sessions.json poll every other panel already pays for.

  import { sessions } from '../stores/index.js';
  import { shortPath } from '../lib/format.js';
  import type { SessionRow } from '../types.js';

  const TOP_N = 8;

  // Label a session by the most human-readable handle we have: the Claude
  // Code project path, then the proxy-observed cwd, then the synthetic
  // first_user_sha8 prefix.
  function label(s: SessionRow): string {
    const proj = s.claudeCode?.projectPath || s.project;
    return proj ? shortPath(proj) : s.id.slice(0, 8);
  }

  function fmt(n: number): string {
    return Math.round(n).toLocaleString('en-US');
  }

  // Sort by tokens saved descending; keep the top N. The bar scale is pinned
  // to the largest saver so the bars are comparable within the panel.
  $: allRows = $sessions.data?.sessions ?? [];
  $: rows = [...allRows]
    .sort((a, b) => (b.tokensSavedEst ?? 0) - (a.tokensSavedEst ?? 0))
    .slice(0, TOP_N);
  $: max = rows.reduce((m, s) => Math.max(m, s.tokensSavedEst ?? 0), 0);

  // Negative savings (a turn where compression net-lost) and zeros get no
  // bar — the red value still tells the story.
  function barPct(v: number): number {
    if (max <= 0 || v <= 0) return 0;
    return (v / max) * 100;
  }
</script>

<div class="status">
  {#if $sessions.loading && allRows.length === 0}
    loading…
  {:else if $sessions.error}
    {$sessions.error}
  {:else}
    {allRows.length} session{allRows.length === 1 ? '' : 's'}
  {/if}
</div>

{#if rows.length > 0}
  <div class="chart">
    {#each rows as s (s.id)}
      <div class="bar-row">
        <div
          class="label"
          title={s.claudeCode?.projectPath || s.project || s.id}
        >{label(s)}</div>
        <div class="track">
          {#if barPct(s.tokensSavedEst ?? 0) > 0}
            <div
              class="fill"
              style="width:max(3px,{barPct(s.tokensSavedEst ?? 0)}%)"
            ></div>
          {/if}
        </div>
        <div class="value" class:neg={(s.tokensSavedEst ?? 0) < 0}>
          {fmt(s.tokensSavedEst ?? 0)}
        </div>
      </div>
    {/each}
  </div>
  <div class="axis">
    input tokens saved (cache-aware) · top {rows.length} of {allRows.length}
  </div>
{:else if !$sessions.loading && !$sessions.error}
  <div class="empty">no sessions yet</div>
{/if}

<style>
  .status {
    margin-bottom: 12px;
    color: #6e7681;
    font-size: 12px;
  }
  .chart {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .bar-row {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 12px;
  }
  .label {
    width: 132px;
    flex: none;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: #c9d1d9;
  }
  .track {
    flex: 1;
    min-width: 0;
    height: 14px;
    background: #21262d;
    border-radius: 3px;
    overflow: hidden;
  }
  .fill {
    height: 100%;
    background: #3fb950;
    border-radius: 3px;
  }
  .value {
    width: 72px;
    flex: none;
    text-align: right;
    font-variant-numeric: tabular-nums;
    color: #3fb950;
  }
  .value.neg {
    color: #f85149;
  }
  .axis {
    margin-top: 12px;
    color: #6e7681;
    font-size: 11px;
  }
  .empty {
    text-align: center;
    color: #6e7681;
    padding: 24px;
    font-size: 12px;
  }
</style>
