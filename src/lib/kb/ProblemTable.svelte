<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import filed from '#lib/data/tickets-4471.json';
  import { problemsOf } from '#lib/kb/problems';
  import { tenantOf } from '#lib/kb/locales';
  import type { Report } from '#lib/kb/causes';

  let { ticket }: { ticket: number } = $props();

  type Filed = { number: number; state: 'open' | 'closed'; reason?: string };
  const tickets = filed.tickets as Record<string, Filed>;
  const link = (n: number) => `https://github.com/${filed.repo}/issues/${n}`;

  const split = $derived(problemsOf(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const withTicket = $derived(split.groups.filter((g) => tickets[g.problem]).length);
  const open = $derived(split.groups.filter((g) => tickets[g.problem]?.state === 'open').length);
  const tally = (n: Record<string, number>) =>
    Object.entries(n)
      .sort((a, b) => b[1] - a[1])
      .map(([k, v]) => `${k} ×${v}`)
      .join(' · ');
</script>

<p class="sum" data-reports={split.merged.length} data-problems={split.groups.length}>
  <strong>{split.merged.length}</strong> reports in #{ticket} ·
  <strong>{split.groups.length}</strong> {split.groups.length === 1 ? 'problem' : 'problems'} ·
  <strong>{withTicket}</strong> with a ticket of their own · <strong>{open}</strong> open
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>came in by</th><th>intake tagged</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      {@const t = tickets[g.problem]}
      <tr data-problem={g.problem} data-reports={g.reports.length}>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td class="tally">{tally(g.channels)}</td>
        <td class="tally">{tally(g.taggedAs)}</td>
        <td class="seen">{g.firstSeen} → {g.lastSeen}</td>
        <td class:none={!t}>
          {#if t}<a href={link(t.number)}>#{t.number}</a> · {t.state}{t.reason ? ` (${t.reason})` : ''}{:else}no ticket yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .tally { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
  .seen { white-space: nowrap; }
</style>
