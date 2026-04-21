# OrchardGuard V2 — Rule Authoring Template

> **Purpose.** Standard shape for a rule file in `lib/rules/<category>/<slug>.ts`. Referenced by planning doc Section 4.9.
> **Status.** Template for use from Phase 2 onward. Expect refinement during implementation.

---

## When to Use

Every rule lives in `lib/rules/<category>/<slug>.ts` with a companion `.test.ts`. Rule categories per Section 4.6:

- `registration`
- `growth_stage`
- `seasonal_max`
- `resistance` (FRAC/IRAC/HRAC rotation)
- `phi`
- `rei`
- `tank_mix`
- `inventory`
- `young_tree_safety`
- `nutrition`
- `weed`
- `coincidence`

## The Interface

Every rule implements:

```typescript
interface Rule {
  slug: string;                    // stable identifier; matches rule_evaluations.rule_slug
  category: RuleCategory;          // from the categories above
  severity: 'hard' | 'advisory' | 'informational';
  appliesTo: ProductCategory[];    // 'pesticide' | 'herbicide' | 'foliar_nutrient' | 'soil_amendment'
  requiredContext: string[];       // context fields this rule needs; assembler skips unused slices
  citations: Citation[];           // sources backing this rule
  evaluate(context: RuleContext, product: Product): RuleEvaluation;
}
```

A rule is a pure function. It does not perform I/O; everything needed is in `context` (planning doc Section 4.3). It does not modify state; the advisor writes `rule_evaluations` rows based on returned evaluations.

## Template

```typescript
// lib/rules/<category>/<slug>.ts

import type { Rule, RuleContext, RuleEvaluation } from '@/lib/rules/types';
import type { Product } from '@/lib/catalogue/types';

export const <slug>Rule: Rule = {
  slug: '<snake_case_rule_slug>',
  category: '<category>',
  severity: '<hard | advisory | informational>',
  appliesTo: ['<product_category>', /* ... */],
  requiredContext: [
    // list context fields this rule reads, e.g.:
    // 'block', 'applications_history', 'recent_model_runs'
  ],
  citations: [
    {
      label: '<short label for display>',
      url: '<optional URL>',
      notes: '<optional inline note>',
    },
  ],

  evaluate(context: RuleContext, product: Product): RuleEvaluation {
    // Early return if rule doesn't apply to this product or this context
    if (/* not applicable */) {
      return {
        ruleSlug: '<snake_case_rule_slug>',
        outcome: 'not_applicable',
        severity: this.severity,
        reason: '<short human-readable reason>',
        details: {},
        citations: this.citations,
      };
    }

    // Core rule logic
    // Read from context, never from I/O
    // Produce a structured outcome

    const detailedResult = /* compute */;

    if (/* passes */) {
      return {
        ruleSlug: '<snake_case_rule_slug>',
        outcome: 'pass',
        severity: this.severity,
        reason: '<human-readable reason>',
        details: { /* structured data for UI and audit */ },
        citations: this.citations,
      };
    }

    if (/* warn condition */) {
      return {
        ruleSlug: '<snake_case_rule_slug>',
        outcome: 'warn',
        severity: this.severity,
        reason: '<human-readable reason>',
        details: { /* structured data */ },
        citations: this.citations,
      };
    }

    // Block
    return {
      ruleSlug: '<snake_case_rule_slug>',
      outcome: 'block',
      severity: this.severity,
      reason: '<human-readable reason — grower will see this>',
      details: { /* structured data */ },
      citations: this.citations,
    };
  },
};
```

## Test File Template

Every rule has a companion test file at `lib/rules/<category>/<slug>.test.ts`.

```typescript
// lib/rules/<category>/<slug>.test.ts

import { describe, it, expect } from 'vitest';
import { <slug>Rule } from './<slug>';
import { buildTestContext, buildTestProduct } from '@/test/rule-helpers';

describe('<slug>Rule', () => {
  it('passes when <positive condition>', () => {
    const context = buildTestContext({ /* setup */ });
    const product = buildTestProduct({ /* setup */ });
    const result = <slug>Rule.evaluate(context, product);
    expect(result.outcome).toBe('pass');
  });

  it('warns when <warn condition>', () => {
    const context = buildTestContext({ /* setup */ });
    const product = buildTestProduct({ /* setup */ });
    const result = <slug>Rule.evaluate(context, product);
    expect(result.outcome).toBe('warn');
    expect(result.details).toMatchObject({ /* expected structure */ });
  });

  it('blocks when <block condition>', () => {
    const context = buildTestContext({ /* setup */ });
    const product = buildTestProduct({ /* setup */ });
    const result = <slug>Rule.evaluate(context, product);
    expect(result.outcome).toBe('block');
  });

  it('returns not_applicable when <irrelevant context>', () => {
    const context = buildTestContext({ /* setup */ });
    const product = buildTestProduct({ /* setup */ });
    const result = <slug>Rule.evaluate(context, product);
    expect(result.outcome).toBe('not_applicable');
  });

  // Edge cases — boundaries on thresholds, null handling, empty arrays
  it('handles <edge case>', () => {
    /* ... */
  });
});
```

Every rule must cover: positive case, at least one warn or block case, not-applicable case (where relevant), and at least one edge case.

## Worked Example — PHI vs Projected Harvest

```typescript
// lib/rules/phi/phi_vs_projected_harvest.ts

import type { Rule, RuleContext, RuleEvaluation } from '@/lib/rules/types';
import type { Product } from '@/lib/catalogue/types';
import { daysBetween, addDays } from '@/lib/date';

export const phiVsProjectedHarvestRule: Rule = {
  slug: 'phi_vs_projected_harvest',
  category: 'phi',
  severity: 'hard',
  appliesTo: ['pesticide', 'herbicide', 'foliar_nutrient'],
  requiredContext: ['block', 'plantings', 'evaluated_at'],
  citations: [
    {
      label: 'Pest Control Products Act, Canada',
      url: 'https://laws-lois.justice.gc.ca/eng/acts/p-9.01/',
      notes: 'PHI is a label-specified legal requirement.',
    },
  ],

  evaluate(context: RuleContext, product: Product): RuleEvaluation {
    const phiDays = getPhiDays(product);

    if (phiDays === null) {
      return {
        ruleSlug: this.slug,
        outcome: 'not_applicable',
        severity: this.severity,
        reason: 'Product has no PHI restriction.',
        details: {},
        citations: this.citations,
      };
    }

    const earliestProjectedHarvest = context.plantings
      .filter(p => p.expectedHarvestDate !== null)
      .map(p => p.expectedHarvestDate!)
      .sort()[0];

    if (!earliestProjectedHarvest) {
      // No harvest date set on any planting — fall back to variety default
      // Handled by phi_vs_plausible_harvest rule (advisory severity) elsewhere
      return {
        ruleSlug: this.slug,
        outcome: 'not_applicable',
        severity: this.severity,
        reason: 'No expected harvest date set for any planting in this block.',
        details: {},
        citations: this.citations,
      };
    }

    const applicationDate = context.evaluated_at;
    const earliestLegalHarvest = addDays(applicationDate, phiDays);
    const gap = daysBetween(applicationDate, earliestProjectedHarvest);

    if (gap >= phiDays) {
      return {
        ruleSlug: this.slug,
        outcome: 'pass',
        severity: this.severity,
        reason: `${gap}-day gap to earliest projected harvest exceeds ${phiDays}-day PHI.`,
        details: {
          phiDays,
          gapDays: gap,
          safetyMarginDays: gap - phiDays,
          earliestProjectedHarvest: earliestProjectedHarvest.toISOString(),
          governingPlanting: /* which planting's harvest date drives this */,
        },
        citations: this.citations,
      };
    }

    return {
      ruleSlug: this.slug,
      outcome: 'block',
      severity: this.severity,
      reason:
        `Applying this product today would violate the ${phiDays}-day PHI. ` +
        `Earliest projected harvest on this block is in ${gap} days ` +
        `(planting: <variety>). PHI requires ≥${phiDays} days.`,
      details: {
        phiDays,
        gapDays: gap,
        shortfallDays: phiDays - gap,
        earliestProjectedHarvest: earliestProjectedHarvest.toISOString(),
        earliestLegalHarvest: earliestLegalHarvest.toISOString(),
      },
      citations: this.citations,
    };
  },
};

function getPhiDays(product: Product): number | null {
  // Pesticide, herbicide, and foliar nutrient categories all have phi_days
  switch (product.category) {
    case 'pesticide': return product.pesticide?.phi_days ?? null;
    case 'herbicide': return product.herbicide?.phi_days ?? null;
    case 'foliar_nutrient': return product.foliar_nutrient?.phi_days ?? null;
    default: return null;
  }
}
```

## Rule Registry

Every rule is registered in `lib/rules/registry.ts`:

```typescript
// lib/rules/registry.ts

import { phiVsProjectedHarvestRule } from './phi/phi_vs_projected_harvest';
// ... import every rule ...

export const allRules: Rule[] = [
  phiVsProjectedHarvestRule,
  // ... every rule, organized by category for readability
];

export function getEnabledRules(): Rule[] {
  return allRules.filter(r => isRuleEnabled(r.slug));
}
```

Registry supports runtime disable via a config table or feature flag (planning doc Section 4.9). Disabled rules do not run; their slugs still appear in audit logs as `not_applicable`.

## Checklist for a New Rule

Before opening the PR:

- [ ] Source file at `lib/rules/<category>/<slug>.ts` with the template structure
- [ ] Test file at `lib/rules/<category>/<slug>.test.ts` with positive, warn/block, not-applicable, and edge-case tests
- [ ] Registry entry in `lib/rules/registry.ts`
- [ ] Citations populated with at least one source
- [ ] Severity appropriate for what the rule protects against (hard for regulatory or irreversible; advisory for best-practice; informational for annotations)
- [ ] `requiredContext` lists every context field the rule reads
- [ ] Reason text reads clearly when shown to a grower
- [ ] Details object carries enough structured data to reconstruct the decision later
- [ ] PR description cites the source of the rule's threshold or logic

## Related Files

- Planning doc Section 4.4 — Rule Interface
- Planning doc Section 4.5 — Outcome Severity
- Planning doc Section 4.6 — Rule Categories
- Planning doc Section 4.9 — Rule Authoring Workflow
- Model citation header template — `model-citation-header-template.md`

---

> **End of template.** Copy, adapt, commit.
