<script setup lang="ts">
defineOptions({ name: "InvestorWhySection" });

const whyCards = [
  {
    title: "The active transport boom.",
    body:
      "Cities restrict cars and promote cycling. GettaShower removes the physical friction—sweat—so sustainable mobility works for professionals.",
    icon: "bike",
  },
  {
    title: "Tourism without downtime.",
    body:
      "Today's traveler maximizes every minute. We unlock dead time after hotel check-out—travel, explore, and refresh before the flight.",
    icon: "travel",
  },
  {
    title: "Climate resilience.",
    body:
      "Heat waves break records every year. On-demand cooling in the city shifts from luxury to health and everyday urban comfort.",
    icon: "climate",
  },
  {
    title: "Scale without bricks.",
    body:
      "We don't build bathrooms. We turn idle capacity at gyms and hotels into net-new revenue through software.",
    icon: "scale",
  },
] as const;

const trends = [
  {
    headline: "From periphery to center: The cycling surge.",
    metric: "+120%",
    metricNote: "Bike modal / lane intensity",
    subtext:
      "Representative increase in cycling uptake across major European capitals (illustrative benchmark: Paris moving from ~5% toward ~11% modal share).",
    takeaway:
      "More bicycles on the street implies exponentially more need for hygiene endpoints at work destinations.",
    viz: "bars",
  },
  {
    headline: "Compact cities: Europe's new standard.",
    metric: "60%",
    metricNote: "Dense metropolitan share",
    subtext:
      "A dominant share of mobile populations concentrates in dense metro regions—policy favors inward growth over greenfield sprawl.",
    takeaway:
      "\"No net land take\" forces central densification. Less private space means maximizing shared services like GettaShower.",
    viz: "donut",
  },
  {
    headline: "Climate and new tourism flows.",
    metric: "3bn",
    metricNote: "EU tourism guest nights (2024)",
    subtext:
      "EU tourism nights remain in the billions while summers break extreme heat records—crowding meets heat stress at scale.",
    takeaway:
      "Tourism saturation plus extreme heat creates the perfect storm for on-demand refresh.",
    viz: "lines",
  },
] as const;
</script>

<template>
  <div class="portal-hero-band portal-hero-band--soft">
    <div class="portal-hero-inner portal-why-trends-inner">
      <section id="the-why" class="why-block" aria-labelledby="why-heading">
        <h2 id="why-heading" class="why-title">Why:</h2>
        <p class="why-lead">
          On-demand infrastructure for new mobility and climate.
        </p>

        <div class="why-grid">
          <article v-for="card in whyCards" :key="card.title" class="why-card">
          <div class="why-icon" aria-hidden="true">
            <!-- Bike -->
            <svg v-if="card.icon === 'bike'" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="11" cy="34" r="7" stroke="currentColor" stroke-width="2.2" />
              <circle cx="37" cy="34" r="7" stroke="currentColor" stroke-width="2.2" />
              <path
                d="M11 34h9l6-14 6 14h6M20 34l7-16 5 8"
                stroke="currentColor"
                stroke-width="2.2"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path d="M27 18l4-4h6" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" />
            </svg>
            <!-- Travel -->
            <svg v-else-if="card.icon === 'travel'" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect x="10" y="16" width="22" height="16" rx="3" stroke="currentColor" stroke-width="2.2" />
              <path d="M14 16v-3a4 4 0 014-4h2a4 4 0 014 4v3" stroke="currentColor" stroke-width="2.2" />
              <path d="M32 22h6l4 4v6h-4" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" />
              <circle cx="38" cy="34" r="3" stroke="currentColor" stroke-width="2.2" />
              <circle cx="22" cy="12" r="4" stroke="currentColor" stroke-width="2.2" />
            </svg>
            <!-- Climate -->
            <svg v-else-if="card.icon === 'climate'" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="24" cy="22" r="9" stroke="currentColor" stroke-width="2.2" stroke-dasharray="3 4" />
              <path
                d="M24 7v3M24 34v3M39 22h-3M12 22H9M34.5 11.5l-2.2 2.2M15.7 34.3l-2.1 2.1M34.5 32.5l-2.2-2.2M15.7 13.7l-2.1-2.1"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
              <path d="M18 38h16" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" />
              <path d="M22 38l2-8h4l2 8" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <!-- Scale -->
            <svg v-else viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M10 38V18l8-6 10 8 10-6v24" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" />
              <path d="M18 38V26l8 6 10-8v14" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" />
              <path d="M14 12l4 3 4-3 4 3 4-3" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
          </div>
            <h3 class="why-card-title">{{ card.title }}</h3>
            <p class="why-card-body">{{ card.body }}</p>
          </article>
        </div>
      </section>

      <section id="market-trends" class="trends-block" aria-labelledby="trends-heading">
        <h2 id="trends-heading" class="trends-title">Market trends</h2>
        <p class="trends-intro">
          Hard signals in a visual layout—investors skim numbers first, then read the angle.
        </p>

        <div class="trends-grid">
          <article v-for="item in trends" :key="item.headline" class="trend-card">
            <div class="trend-copy">
              <h3 class="trend-headline">{{ item.headline }}</h3>
              <p class="trend-metric" :data-note="item.metricNote">{{ item.metric }}</p>
              <p class="trend-sub">{{ item.subtext }}</p>
              <p class="trend-takeaway"><span>Angle:</span> {{ item.takeaway }}</p>
            </div>
            <div class="trend-viz" aria-hidden="true">
            <!-- Ascending bars -->
              <svg v-if="item.viz === 'bars'" viewBox="0 0 120 72" class="mini-chart">
                <rect x="8" y="48" width="14" height="18" rx="2" class="bar bar-a" />
                <rect x="30" y="38" width="14" height="28" rx="2" class="bar bar-b" />
                <rect x="52" y="28" width="14" height="38" rx="2" class="bar bar-c" />
                <rect x="74" y="18" width="14" height="48" rx="2" class="bar bar-d" />
                <rect x="96" y="8" width="14" height="58" rx="2" class="bar bar-e" />
              </svg>
              <!-- Donut 60% -->
              <svg v-else-if="item.viz === 'donut'" viewBox="0 0 120 72" class="mini-chart donut-svg">
                <circle cx="44" cy="36" r="26" class="donut-track" />
                <circle cx="44" cy="36" r="26" class="donut-fill" />
                <g class="heat-grid">
                  <rect v-for="i in 16" :key="i" :x="76 + ((i - 1) % 4) * 10" :y="14 + Math.floor((i - 1) / 4) * 12" width="7" height="8" rx="1" :class="['heat-cell', i <= 10 ? 'hot' : '']" />
                </g>
              </svg>
              <!-- Dual lines -->
              <svg v-else viewBox="0 0 120 72" class="mini-chart">
                <path d="M8 52 C28 48 32 28 52 22 S88 18 112 12" class="line-a" />
                <path d="M8 58 C24 56 40 40 58 34 S92 24 112 20" class="line-b" />
                <line x1="8" y1="62" x2="112" y2="62" class="baseline" />
              </svg>
            </div>
          </article>
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped>
.portal-why-trends-inner {
  display: flex;
  flex-direction: column;
  gap: clamp(2.25rem, 5vw, 3.35rem);
}

.why-block,
.trends-block {
  scroll-margin-top: var(--portal-scroll-anchor);
}

.why-title {
  margin: 0 0 0.35rem;
  font-size: clamp(1.65rem, 3vw, 2.35rem);
  font-weight: 850;
  line-height: 1.12;
  letter-spacing: -0.02em;
  color: var(--gs-text);
}

.why-lead {
  margin: 0 0 1.25rem;
  font-size: clamp(1.05rem, 1.6vw, 1.22rem);
  font-weight: 600;
  color: var(--gs-text-soft);
  max-width: 52ch;
  line-height: 1.45;
}

.why-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 1.35rem 1.5rem;
}

.why-card {
  background: transparent;
  border: none;
  border-radius: 0;
  padding: 0 0 0 1rem;
  border-left: 3px solid color-mix(in srgb, var(--gs-primary) 35%, transparent);
  box-shadow: none;
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
  min-height: 100%;
}

.why-icon {
  width: 3rem;
  height: 3rem;
  border-radius: 14px;
  background: rgba(26, 43, 112, 0.06);
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--gs-primary);
}

.why-icon svg {
  width: 1.85rem;
  height: 1.85rem;
}

.why-card-title {
  margin: 0;
  font-size: 1.08rem;
  font-weight: 800;
  line-height: 1.22;
  color: var(--gs-text);
  letter-spacing: -0.01em;
}

.why-card-body {
  margin: 0;
  font-size: 0.94rem;
  line-height: 1.48;
  color: var(--gs-text-soft);
  flex: 1;
}

.trends-title {
  margin: 0 0 0.35rem;
  font-size: clamp(1.45rem, 2.4vw, 1.95rem);
  font-weight: 850;
  color: var(--gs-text);
  letter-spacing: -0.02em;
}

.trends-intro {
  margin: 0 0 1.1rem;
  font-size: 1rem;
  color: var(--gs-text-soft);
  max-width: 56ch;
  line-height: 1.45;
}

.trends-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1.35rem 1.75rem;
}

.trend-card {
  background: transparent;
  border: none;
  border-radius: 0;
  padding: 0;
  border-bottom: none;
  display: grid;
  grid-template-columns: 1fr minmax(112px, 38%);
  gap: 1rem;
  align-items: stretch;
  box-shadow: none;
  min-width: 0;
}

.trend-headline {
  margin: 0 0 0.45rem;
  font-size: 1.02rem;
  font-weight: 800;
  line-height: 1.25;
  color: var(--gs-text);
}

.trend-metric {
  margin: 0 0 0.35rem;
  font-size: clamp(2.1rem, 4vw, 2.65rem);
  font-weight: 900;
  letter-spacing: -0.03em;
  color: var(--gs-primary);
  line-height: 1;
}

.trend-metric::after {
  content: attr(data-note);
  display: block;
  margin-top: 0.35rem;
  font-size: 0.68rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--gs-text-muted);
}

.trend-sub {
  margin: 0 0 0.65rem;
  font-size: 0.88rem;
  line-height: 1.45;
  color: var(--gs-text-soft);
}

.trend-takeaway {
  margin: 0;
  font-size: 0.84rem;
  line-height: 1.42;
  color: var(--gs-text-soft);
  padding: 0.55rem 0;
  border-radius: 0;
  background: transparent;
  border: none;
  border-top: 1px solid color-mix(in srgb, var(--gs-border) 55%, transparent);
  padding-top: 0.65rem;
  margin-top: 0.15rem;
}

.trend-takeaway span {
  font-weight: 800;
  color: var(--gs-primary);
  margin-right: 0.25rem;
}

.trend-viz {
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.55);
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 108px;
  padding: 0.45rem;
}

.mini-chart {
  width: 100%;
  height: auto;
  max-height: 88px;
}

.bar {
  fill: color-mix(in srgb, var(--gs-primary) 45%, white);
  opacity: 0.85;
}

.bar-e {
  fill: var(--gs-primary);
}

.bar-d {
  fill: color-mix(in srgb, var(--gs-primary) 72%, white);
}

.bar-c {
  fill: color-mix(in srgb, var(--gs-primary) 58%, white);
}

.donut-track {
  fill: none;
  stroke: var(--gs-border);
  stroke-width: 10;
}

.donut-fill {
  fill: none;
  stroke: var(--gs-primary);
  stroke-width: 10;
  stroke-dasharray: 163.36;
  stroke-dashoffset: 65.35;
  stroke-linecap: round;
  transform: rotate(-90deg);
  transform-origin: 44px 36px;
}

.heat-grid {
  opacity: 0.95;
}

.heat-cell {
  fill: var(--gs-bg-soft);
}

.heat-cell.hot {
  fill: var(--gs-primary);
  opacity: 0.35;
}

.heat-cell.hot:nth-child(odd) {
  opacity: 0.55;
}

.line-a {
  fill: none;
  stroke: var(--gs-primary);
  stroke-width: 2.5;
  stroke-linecap: round;
}

.line-b {
  fill: none;
  stroke: #f3a041;
  stroke-width: 2.5;
  stroke-linecap: round;
  opacity: 0.9;
}

.baseline {
  stroke: var(--gs-border);
  stroke-width: 1;
}

@media (max-width: 1080px) {
  .why-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .trends-grid {
    grid-template-columns: 1fr;
  }

  .trend-card {
    grid-template-columns: 1fr;
    padding-bottom: 1.35rem;
    margin-bottom: 1.35rem;
    border-bottom: 1px solid color-mix(in srgb, var(--gs-border) 55%, transparent);
  }

  .trends-grid .trend-card:last-child {
    border-bottom: none;
    margin-bottom: 0;
    padding-bottom: 0;
  }

  .trend-viz {
    min-height: 96px;
    max-width: none;
  }
}

@media (max-width: 560px) {
  .why-grid {
    grid-template-columns: 1fr;
  }

  .trend-viz {
    max-width: none;
  }
}
</style>
