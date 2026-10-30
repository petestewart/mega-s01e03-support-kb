<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket, repo }: { ticket: number; repo: string } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  const moved = $derived(groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0));

  // The one-day moves stay in the data as `parser`; here they count as their own problem.
  const rows = $derived([
    { issue: 1, problem: 'shared parser reads dates month first', reports: count('parser') - moved, outcome: 'fixed: parse dd/mm for eu locales' },
    { issue: 2, problem: 'booking-v3 hint says MM/DD/YYYY', reports: count('hint'), outcome: "won't fix: booking-v3 form retired" },
    { issue: 3, problem: 'dates move by one day (cause not proven)', reports: moved, outcome: 'open: no verdict from engineering' },
  ]);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{rows.length}</strong> problems ·
  CauseTable shows <strong>{count('parser')}</strong> for <code>parser</code> because it counts the
  <strong>{moved}</strong> one-day moves there
</p>

<table>
  <thead><tr><th>ticket</th><th>problem</th><th>reports</th><th>outcome</th></tr></thead>
  <tbody>
    {#each rows as r (r.issue)}
      <tr>
        <td><a href="{repo}/issues/{r.issue}">{r.issue}</a></td>
        <td>{r.problem}</td>
        <td>{r.reports}</td>
        <td>{r.outcome}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
