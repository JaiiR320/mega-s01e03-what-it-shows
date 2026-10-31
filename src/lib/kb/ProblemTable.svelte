<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Problem = { name: string; issue: number };
  let { ticket, repo, parser, hint, oneDay }: { ticket: number; repo: string; parser: Problem; hint: Problem; oneDay: Problem } = $props();

  const rows = $derived(uniqueById(reports as Report[]));
  const merged = $derived(rows.filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const swapped = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));

  // The swaps split by cause.ts. The one-day moves are not swaps, so they leave the cause they were filed under.
  const problems = $derived([
    ...groupByCause(swapped, tenantOf).map((g) => ({ ...(g.cause === 'hint' ? hint : parser), reports: g.reports })),
    { ...oneDay, reports: moved },
  ]);
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{problems.length}</strong> problems ·
  {(reports as Report[]).length - rows.length} repeated export rows left out
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each problems as p (p.issue)}
      <tr>
        <td>{p.name}</td>
        <td>{p.reports.length}</td>
        <td>{new Set(p.reports.map((r) => r.tenant)).size}</td>
        <td>{seen(p.reports)}</td>
        <td><a href="https://github.com/{repo}/issues/{p.issue}">{repo}#{p.issue}</a></td>
      </tr>
    {/each}
  </tbody>
</table>
