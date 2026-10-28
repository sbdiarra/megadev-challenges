<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { groupByProblem, type ProblemGroup } from '#lib/kb/problems';

  // issues: problem → where its ticket lives, e.g. { parser: 'https://…/issues/1' }
  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<ProblemGroup['problem'], string>> } = $props();

  const split = $derived(groupByProblem(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const counted = $derived(split.groups.reduce((n, g) => n + g.reports.length, 0));
</script>

<p class="sum">
  <strong>{split.merged}</strong> unique reports in #{ticket} ·
  <strong>{split.groups.length}</strong> problems · {counted === split.merged ? 'each report counted once' : `${split.merged - counted} not counted`}
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if issues[g.problem]}<a href={issues[g.problem]}>{issues[g.problem]?.replace(/^.*\/issues\//, 'issue #')}</a>{:else}none yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
