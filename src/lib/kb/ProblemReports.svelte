<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { problemsOf, type Problem } from '#lib/kb/problems';
  import { tenantOf } from '#lib/kb/locales';
  import type { Report } from '#lib/kb/causes';

  // Every report of one problem, as the reporter wrote it.
  let { ticket, problem }: { ticket: number; problem: Problem } = $props();

  const rows = $derived(
    problemsOf(reports as Report[], ticket, tenantOf, notes.oneDayMove)
      .groups.find((g) => g.problem === problem)?.reports ?? [],
  );
</script>

<table>
  <thead><tr><th>report</th><th>received</th><th>tenant</th><th>what they wrote</th></tr></thead>
  <tbody>
    {#each rows as r (r.id)}
      <tr>
        <td><code>{r.id}</code></td>
        <td>{r.receivedAt.slice(0, 10)}</td>
        <td>{tenantOf(r)?.name ?? r.tenant} <span class="loc">{tenantOf(r)?.locale ?? ''}</span></td>
        <td>{r.body}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .loc { color: var(--muted); font: 0.8em var(--mono); }
</style>
