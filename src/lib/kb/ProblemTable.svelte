<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { exportDuplicates, groupByProblem, type Problem } from '#lib/kb/problems';
  import { tenantOf } from '#lib/kb/locales';

  type Ticket = { issue: number; url: string; outcome: string };
  let { ticket, tickets }: { ticket: number; tickets: Partial<Record<Problem, Ticket>> } = $props();

  const raw = reports as Report[];
  const merged = $derived(uniqueById(raw).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByProblem(merged, tenantOf, notes.oneDayMove));
  const placed = $derived(groups.reduce((n, g) => n + g.reports.length, 0));
  const counts = (m: Record<string, number>) =>
    Object.entries(m)
      .sort((a, b) => b[1] - a[1])
      .map(([k, n]) => `${k} ×${n}`)
      .join(' · ');
  const share = (n: number) => `${Math.round((n / merged.length) * 100)}%`;
  const ids = (g: { reports: Report[] }) => g.reports.map((r) => r.id).join(', ');
</script>

<p class="sum">
  <strong>{raw.length}</strong> rows in the export · <strong>{exportDuplicates(raw)}</strong> repeated ids ·
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> {groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>share</th><th>tenants</th><th>way in</th><th>intake tagged as</th><th>seen</th><th>ticket</th><th>outcome</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      {@const t = g.problem === 'unplaced' ? undefined : tickets[g.problem]}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{share(g.reports.length)}</td>
        <td>{g.tenants}</td>
        <td class="tags">{counts(g.channels)}</td>
        <td class="tags">{counts(g.taggedAs)}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if t}<a href={t.url}>#{t.issue}</a>{:else}none{/if}</td>
        <td>{t?.outcome ?? ""}</td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  {placed} of {merged.length} reports placed{placed === merged.length ? '; every report sits in exactly one problem.' : '.'}
</p>

{#each groups as g (g.problem)}
  {#if g.reports.length <= 12}
    <p class="note"><code>{g.problem}</code> reports: {ids(g)}</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .tags { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
