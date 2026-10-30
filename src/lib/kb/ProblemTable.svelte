<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<Cause, string>> } = $props();

  const rows = reports as Report[];
  const merged = $derived(uniqueById(rows).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const fmt = (n: Record<string, number>) =>
    Object.entries(n)
      .sort((a, b) => b[1] - a[1])
      .map(([k, v]) => `${k} ×${v}`)
      .join(' · ');
  const channels = (rs: Report[]) => fmt(rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.channel]: (n[r.channel] ?? 0) + 1 }), {}));
  const issue = (cause: string) => issues[cause as Cause];
</script>

<p class="sum">
  The export has <strong>{rows.length}</strong> rows; <strong>{rows.length - uniqueById(rows).length}</strong> are
  repeats from overlapping pages. <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> {groups.length === 1 ? 'problem' : 'problems'}.
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>intake tagged as</th><th>tenants</th><th>channels</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td><strong>{g.reports.length}</strong></td>
        <td class="small">{fmt(g.taggedAs)}</td>
        <td class="small">{g.tenants}</td>
        <td class="small">{channels(g.reports)}</td>
        <td class="small">{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if issue(g.cause)}<a href={issue(g.cause)}>{issue(g.cause)?.replace(/.*\/issues\//, '#')}</a>{:else}—{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="small">{moved} reports read as a one-day move; filed under <code>{g.cause}</code> by tenant.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.05rem; }
  .small { color: var(--muted); font-size: 0.85em; }
</style>
