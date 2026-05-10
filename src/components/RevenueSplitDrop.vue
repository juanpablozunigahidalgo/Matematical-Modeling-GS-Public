<script setup lang="ts">
import { computed } from "vue";

defineOptions({ name: "RevenueSplitDrop" });

const props = withDefaults(
  defineProps<{
    platformTakePct: number;
    hostTakePct: number;
    cmPct: number;
    adsPct: number;
    /** Numeric shower price (EUR) — used to show each stakeholder’s € per shower */
    priceEur?: number;
    priceLabel: string;
  }>(),
  {
    priceEur: 0,
  },
);

function formatShowerEur(value: number): string {
  return new Intl.NumberFormat("en-GB", {
    style: "currency",
    currency: "EUR",
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(value);
}

const CX = 100;
const CY = 100;
const R = 84;

type Seg = {
  key: string;
  label: string;
  svgLabel: string;
  sub: string;
  pct: number;
  fill: string;
};

const palette = {
  cac: { fill: "#e4ebfb" },
  cm: { fill: "#b8c9ef" },
  platform: { fill: "#1a2b70" },
  host: { fill: "#3d8f6a" },
};

const segmentDefs = computed<Seg[]>(() => {
  const safe = (n: unknown) => (Number.isFinite(Number(n)) ? Number(n) : 0);
  return [
    {
      key: "cac",
      label: "CAC",
      svgLabel: "CAC",
      sub: "customer acquisition",
      pct: safe(props.adsPct),
      ...palette.cac,
    },
    {
      key: "cm",
      label: "Country manager",
      svgLabel: "CM",
      sub: "local distribution",
      pct: safe(props.cmPct),
      ...palette.cm,
    },
    {
      key: "platform",
      label: "GettaShower",
      svgLabel: "GS",
      sub: "platform take",
      pct: safe(props.platformTakePct),
      ...palette.platform,
    },
    {
      key: "host",
      label: "Host",
      svgLabel: "Host",
      sub: "partner venues",
      pct: safe(props.hostTakePct),
      ...palette.host,
    },
  ];
});

const splitTotal = computed(() => segmentDefs.value.reduce((s, x) => s + x.pct, 0));

const legendSegments = computed(() => {
  const price = Number.isFinite(props.priceEur) ? props.priceEur : 0;
  return segmentDefs.value.map((s) => ({
    ...s,
    euroFormatted: formatShowerEur(price * (s.pct / 100)),
  }));
});

function polar(cx: number, cy: number, r: number, angle: number) {
  return {
    x: cx + r * Math.cos(angle),
    y: cy + r * Math.sin(angle),
  };
}

/** Wedge from center: a0 → a1 radians (clockwise positive from +x). */
function wedgePath(cx: number, cy: number, r: number, a0: number, a1: number): string {
  const p0 = polar(cx, cy, r, a0);
  const p1 = polar(cx, cy, r, a1);
  const delta = a1 - a0;
  const large = Math.abs(delta) > Math.PI ? 1 : 0;
  const sweep = delta >= 0 ? 1 : 0;
  return `M ${cx} ${cy} L ${p0.x.toFixed(4)} ${p0.y.toFixed(4)} A ${r} ${r} 0 ${large} ${sweep} ${p1.x.toFixed(4)} ${p1.y.toFixed(4)} Z`;
}

const pieLayout = computed(() => {
  const total = splitTotal.value;
  const slices: Array<
    Seg & {
      d: string;
      midAngle: number;
      showLabel: boolean;
      fullCircle: boolean;
    }
  > = [];

  if (total <= 0) {
    return { slices, valid: true, total: 0 };
  }

  /** Start at top (−90°), go clockwise. */
  let angle = -Math.PI / 2;

  for (const seg of segmentDefs.value) {
    const frac = seg.pct / total;
    const span = frac * 2 * Math.PI;
    if (seg.pct <= 0 || span < 1e-6) {
      continue;
    }
    const a0 = angle;
    const a1 = angle + span;
    const midAngle = (a0 + a1) / 2;
    const showLabel = seg.pct >= 10;

    if (span >= 2 * Math.PI - 1e-4) {
      slices.push({
        ...seg,
        d: "",
        midAngle: -Math.PI / 2,
        showLabel: true,
        fullCircle: true,
      });
      angle = a1;
      continue;
    }

    const d = wedgePath(CX, CY, R, a0, a1);
    slices.push({ ...seg, d, midAngle, showLabel, fullCircle: false });
    angle = a1;
  }

  return {
    slices,
    valid: Math.abs(total - 100) < 0.05,
    total,
  };
});

const ariaLabel = computed(() => {
  const price = Number.isFinite(props.priceEur) ? props.priceEur : 0;
  const { slices, total } = pieLayout.value;
  if (!slices.length || total <= 0) {
    return "Revenue split chart: no percentages to display.";
  }
  const parts = segmentDefs.value
    .map((s) => {
      const eur = formatShowerEur(price * (s.pct / 100));
      return `${s.label} ${s.pct.toFixed(1)}%, ${eur} per shower`;
    })
    .join(", ");
  return `Revenue split per shower: ${parts}. Total ${total.toFixed(1)} percent.`;
});

const labelRadius = R * 0.58;
</script>

<template>
  <section class="revenue-split-drop" aria-labelledby="revenue-split-drop-title">
    <div class="revenue-split-drop-head">
      <h2 id="revenue-split-drop-title" class="revenue-split-drop-title">Shower Cut

      </h2>
      <p class="revenue-split-drop-lead">
        Here you can see how each shower payment splits. Find the percentage breakdown below. 
      </p>
    </div>

    <p v-if="pieLayout.slices.length && !pieLayout.valid" class="revenue-split-drop-warn" role="status">
      Splits currently sum to <strong>{{ pieLayout.total.toFixed(2) }}%</strong> — adjust assumptions until the total is 100% to match the model.
    </p>

    <div class="revenue-split-drop-body">
      <svg
        class="revenue-split-drop__svg revenue-split-drop__svg--pie"
        viewBox="0 0 200 200"
        xmlns="http://www.w3.org/2000/svg"
        role="img"
        :aria-label="ariaLabel"
      >
        <title>{{ ariaLabel }}</title>

        <circle :cx="CX" :cy="CY" :r="R" class="revenue-split-pie__rim" fill="none" />

        <template v-if="pieLayout.slices.length">
          <template v-for="s in pieLayout.slices" :key="s.key">
            <circle
              v-if="s.fullCircle"
              :cx="CX"
              :cy="CY"
              :r="R"
              :fill="s.fill"
              class="revenue-split-pie__slice"
              stroke="#ffffff"
              stroke-width="1.25"
            />
            <path
              v-else
              :d="s.d"
              :fill="s.fill"
              class="revenue-split-pie__slice"
              stroke="#ffffff"
              stroke-width="1.25"
              stroke-linejoin="round"
            />
          </template>
          <g v-for="s in pieLayout.slices" :key="'lbl-' + s.key">
            <text
              v-if="s.showLabel"
              :x="CX + labelRadius * Math.cos(s.midAngle)"
              :y="CY + labelRadius * Math.sin(s.midAngle)"
              text-anchor="middle"
              dominant-baseline="central"
              class="revenue-split-pie__slice-label"
              :class="s.key === 'cac' || s.key === 'cm' ? 'revenue-split-pie__slice-label--dark' : 'revenue-split-pie__slice-label--light'"
            >
              {{ Math.round(s.pct) }}%
            </text>
          </g>
        </template>

        <text v-else :x="CX" :y="CY" text-anchor="middle" dominant-baseline="central" class="revenue-split-drop__empty">
          Enter splits
        </text>
      </svg>

      <ul class="revenue-split-drop-legend" aria-label="Split legend with amounts per shower">
        <li v-for="b in legendSegments" :key="b.key" class="revenue-split-drop-legend__item">
          <span class="revenue-split-drop-legend__swatch" :style="{ background: b.fill }" />
          <span class="revenue-split-drop-legend__text">
            <span class="revenue-split-drop-legend__main">
              <span class="revenue-split-drop-legend__labels">
                <strong>{{ b.label }}</strong>
                <span class="revenue-split-drop-legend__pct">{{ b.pct }}%</span>
              </span>
              <span class="revenue-split-drop-legend__euro" :title="`${b.euroFormatted} per shower at ${priceLabel} shower price`">{{
                b.euroFormatted
              }}</span>
            </span>
            <span class="revenue-split-drop-legend__sub">{{ b.sub }}</span>
          </span>
        </li>
      </ul>
    </div>

    <p class="revenue-split-drop-price-line">
      User pays <strong>{{ priceLabel }}</strong> per shower.
    </p>
    <p class="revenue-split-drop-lead revenue-split-drop-summary">
      The revenue from each shower is distributed among the various stakeholders according to the established payment split.
    </p>
  </section>
</template>

<style scoped>
.revenue-split-drop {
  margin: 0 0 1.25rem;
  padding: 1.15rem 1.15rem 1.05rem;
  border-radius: var(--gs-radius-lg, 16px);
  border: 1px solid var(--gs-border, #d0d8eb);
  background: linear-gradient(180deg, var(--gs-bg-soft, #eef1f8) 0%, var(--gs-bg, #ffffff) 42%);
  box-shadow: var(--gs-shadow-card, 0 8px 28px rgba(26, 43, 112, 0.07));
}

.revenue-split-drop-head {
  margin-bottom: 0.85rem;
}

.revenue-split-drop-title {
  margin: 0 0 0.35rem;
  font-size: clamp(1.2rem, 2vw, 1.45rem);
  font-weight: 800;
  letter-spacing: -0.02em;
  color: var(--gs-text, #1a2b70);
  line-height: 1.15;
}

.revenue-split-drop-lead {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.45;
  color: var(--gs-text-soft, #4a5f9e);
  max-width: 62ch;
}

.revenue-split-drop-warn {
  margin: 0 0 0.75rem;
  padding: 0.5rem 0.65rem;
  border-radius: 10px;
  background: #fff8e6;
  border: 1px solid #e8c966;
  color: #6b5400;
  font-size: 0.84rem;
  line-height: 1.4;
}

.revenue-split-drop-body {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 1rem 1.5rem;
}

.revenue-split-drop__svg {
  width: min(200px, 52vw);
  height: auto;
  flex-shrink: 0;
  display: block;
}

.revenue-split-drop__svg--pie {
  aspect-ratio: 1;
}

.revenue-split-pie__rim {
  stroke: color-mix(in srgb, var(--gs-primary, #1a2b70) 28%, #ffffff);
  stroke-width: 2;
}

.revenue-split-pie__slice {
  outline: none;
}

.revenue-split-pie__slice-label {
  font-size: 13px;
  font-weight: 800;
  letter-spacing: 0.02em;
  pointer-events: none;
}

.revenue-split-pie__slice-label--light {
  fill: #ffffff;
}

.revenue-split-pie__slice-label--dark {
  fill: var(--gs-text, #1a2b70);
}

.revenue-split-drop__empty {
  fill: var(--gs-text-muted, #6b7db8);
  font-size: 11px;
  font-weight: 600;
}

.revenue-split-drop-legend {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  gap: 0.45rem;
  min-width: min(100%, 280px);
  flex: 1;
}

.revenue-split-drop-legend__item {
  display: flex;
  align-items: flex-start;
  gap: 0.55rem;
  font-size: 0.82rem;
  line-height: 1.35;
  color: var(--gs-text-soft, #4a5f9e);
}

.revenue-split-drop-legend__swatch {
  width: 0.65rem;
  height: 0.65rem;
  border-radius: 3px;
  flex-shrink: 0;
  margin-top: 0.28rem;
  box-shadow: inset 0 0 0 1px rgba(26, 43, 112, 0.12);
}

.revenue-split-drop-legend__text {
  display: flex;
  flex-direction: column;
  gap: 0.12rem;
  flex: 1;
  min-width: 0;
}

.revenue-split-drop-legend__text strong {
  color: var(--gs-text, #1a2b70);
  font-weight: 800;
}

.revenue-split-drop-legend__main {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 0.5rem;
  width: 100%;
}

.revenue-split-drop-legend__labels {
  display: flex;
  flex-wrap: wrap;
  align-items: baseline;
  gap: 0.25rem 0.5rem;
  min-width: 0;
}

.revenue-split-drop-legend__pct {
  font-weight: 700;
  color: var(--gs-primary, #1a2b70);
}

.revenue-split-drop-legend__euro {
  font-weight: 800;
  font-variant-numeric: tabular-nums;
  font-size: 0.84rem;
  color: var(--gs-text, #1a2b70);
  flex-shrink: 0;
}

.revenue-split-drop-legend__sub {
  width: 100%;
  font-size: 0.76rem;
  color: var(--gs-text-muted, #6b7db8);
}

.revenue-split-drop-price-line {
  margin: 1rem 0 0;
  text-align: left;
  font-size: 0.9rem;
  line-height: 1.45;
  font-weight: 400;
  color: var(--gs-text-soft, #4a5f9e);
  max-width: 62ch;
}

.revenue-split-drop-price-line strong {
  color: var(--gs-text, #1a2b70);
  font-weight: 700;
}

.revenue-split-drop-summary {
  margin-top: 0.5rem;
}

@media (min-width: 640px) {
  .revenue-split-drop-body {
    justify-content: space-between;
  }

  .revenue-split-drop__svg {
    width: min(220px, 36vw);
  }
}
</style>
