<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";
import csvRaw from "./data/cities.csv?raw";
import hostFacilitiesRaw from "./data/host-facilities.csv?raw";

type CityRef = {
  city: string;
  population: number;
  averageMonthlyTourists: number;
};

type Scenario = {
  price: number;
  platformTakePct: number;
  hostTakePct: number;
  cmPct: number;
  adsPct: number;
  hostCapPct: number;
  rampPctMonth0: number;
  rampPctMonth8: number;
  rampPctMonth16: number;
  rampPctMonth24: number;
  showersPerHostDay: number;
  employees: number;
  avgSalary: number;
  fixedOps: number;
  initialInvestment: number;
  initialMonth: string;
  months: number;
};

type Row = {
  monthLabel: string;
  activeCities: string;
  hostsTotal: number;
  sessions: number;
  gmv: number;
  appRevenue: number;
  costs: number;
  net: number;
  cumulativeCash: number;
};

type HostFacilityRef = {
  city: string;
  estimatedTotalFacilities: number;
};

const view = ref<"simulator" | "reference" | "math">("simulator");
const page = ref<"portal" | "roi-model">("portal");

const defaultScenario: Scenario = {
  price: 8,
  platformTakePct: 20,
  hostTakePct: 70,
  cmPct: 6,
  adsPct: 4,
  hostCapPct: 15,
  rampPctMonth0: 1,
  rampPctMonth8: 40,
  rampPctMonth16: 80,
  rampPctMonth24: 100,
  showersPerHostDay: 3,
  employees: 2,
  avgSalary: 6000,
  fixedOps: 1500,
  initialInvestment: 350000,
  initialMonth: "2026-06",
  months: 24,
};
const AVG_MONTH_LENGTH_DAYS = 30.4;

const scenario = ref<Scenario>({ ...defaultScenario });
const scenarioDraft = ref<Scenario>({ ...defaultScenario });
const lastRunAt = ref<string>("");
const justRan = ref(false);

type CityRollout = {
  launchOffset: number;
};

const rollout: Record<string, CityRollout> = {
  barcelona: { launchOffset: 0 },
  rome: { launchOffset: 4 },
  madrid: { launchOffset: 8 },
  milan: { launchOffset: 12 },
  paris: { launchOffset: 16 },
  munich: { launchOffset: 20 },
  athens: { launchOffset: 24 },
};

const cityData = computed<CityRef[]>(() => {
  const lines = csvRaw.trim().split("\n");
  const dataLines = lines.slice(1);
  return dataLines.map((line) => {
    const [city, population, avgTourists] = line.split(",");
    return {
      city: city.trim(),
      population: Number(population),
      averageMonthlyTourists: Number(avgTourists),
    };
  });
});

const hostFacilitiesData = computed<HostFacilityRef[]>(() => {
  const lines = hostFacilitiesRaw.trim().split("\n");
  const dataLines = lines.slice(1);
  return dataLines.map((line) => {
    const [city, total] = line.split(",");
    return {
      city: city.trim(),
      estimatedTotalFacilities: Number(total),
    };
  });
});

const hostCapByCity = computed<Record<string, number>>(() => {
  const map: Record<string, number> = {};
  hostFacilitiesData.value.forEach((entry) => {
    map[entry.city.toLowerCase()] = Math.round(entry.estimatedTotalFacilities * (scenario.value.hostCapPct / 100));
  });
  return map;
});

const shareError = computed(() => {
  const total =
    scenarioDraft.value.platformTakePct +
    scenarioDraft.value.hostTakePct +
    scenarioDraft.value.cmPct +
    scenarioDraft.value.adsPct;
  return Math.abs(total - 100) > 0.001
    ? `Total of Host + Platform + CAC + Country manager is ${total.toFixed(2)}% and should be 100%, pplease review.`
    : "";
});

const rampError = computed(() => {
  const m0 = scenarioDraft.value.rampPctMonth0;
  const m8 = scenarioDraft.value.rampPctMonth8;
  const m16 = scenarioDraft.value.rampPctMonth16;
  const m24 = scenarioDraft.value.rampPctMonth24;
  const inRange = [m0, m8, m16, m24].every((v) => v >= 0 && v <= 100);
  if (!inRange) return "Host ramp milestones must be between 0% and 100%.";
  if (!(m0 <= m8 && m8 <= m16 && m16 <= m24)) {
    return "Host ramp milestones must be non-decreasing (month 0 <= 8 <= 16 <= 24).";
  }
  return "";
});

const hostCapError = computed(() => {
  const v = scenarioDraft.value.hostCapPct;
  return v < 0 || v > 100 ? "Host cap used (%) must be between 0 and 100." : "";
});

const initialMonthError = computed(() => {
  const v = scenarioDraft.value.initialMonth;
  return /^\d{4}-\d{2}$/.test(v) ? "" : "Initial date must be a valid month (YYYY-MM).";
});

const runError = computed(() => shareError.value || rampError.value || hostCapError.value || initialMonthError.value);
const hasPendingChanges = computed(() => JSON.stringify(scenarioDraft.value) !== JSON.stringify(scenario.value));

function runModel() {
  if (runError.value) return;
  if (!hasPendingChanges.value) return;
  scenario.value = { ...scenarioDraft.value };
  lastRunAt.value = new Date().toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit", second: "2-digit" });
  justRan.value = true;
  setTimeout(() => {
    justRan.value = false;
  }, 900);
}

function monthLabel(index: number): string {
  const [yearRaw, monthRaw] = scenario.value.initialMonth.split("-");
  const year = Number(yearRaw);
  const month = Number(monthRaw) - 1;
  const safeYear = Number.isFinite(year) ? year : 2026;
  const safeMonth = Number.isFinite(month) ? month : 5;
  const d = new Date(safeYear, safeMonth + index, 1);
  return d.toLocaleDateString("en-GB", { month: "short", year: "2-digit" });
}

const rows = computed<Row[]>(() => {
  const s = scenario.value;
  const platformTake = s.platformTakePct / 100;

  let cumulativeCash = -s.initialInvestment;
  const output: Row[] = [];

  function hostsInCity(cityName: string, monthIdx: number): number {
    const key = cityName.toLowerCase();
    const r = rollout[key];
    if (!r) return 0;
    if (monthIdx < r.launchOffset) return 0;
    const localMonth = monthIdx - r.launchOffset;
    const cap = hostCapByCity.value[key] ?? 0;
    if (cap <= 0) return 0;
    const m0 = s.rampPctMonth0 / 100;
    const m8 = s.rampPctMonth8 / 100;
    const m16 = s.rampPctMonth16 / 100;
    const m24 = s.rampPctMonth24 / 100;
    // Piecewise host ramp by city using user-defined milestones.
    let adoptionFactor = 1;
    if (localMonth <= 8) {
      const progress = localMonth / 8;
      adoptionFactor = m0 + (m8 - m0) * progress;
    } else if (localMonth <= 16) {
      const progress = (localMonth - 8) / 8;
      adoptionFactor = m8 + (m16 - m8) * progress;
    } else if (localMonth <= 24) {
      const progress = (localMonth - 16) / 8;
      adoptionFactor = m16 + (m24 - m16) * progress;
    } else {
      adoptionFactor = m24;
    }
    return Math.round(cap * adoptionFactor);
  }

  for (let m = 0; m < Math.max(1, Math.round(s.months)); m += 1) {
    let monthlySessions = 0;
    let monthlyHosts = 0;
    const activeCities: string[] = [];

    cityData.value.forEach((city) => {
      const hosts = hostsInCity(city.city, m);
      monthlyHosts += hosts;
      monthlySessions += hosts * s.showersPerHostDay * AVG_MONTH_LENGTH_DAYS;
      if (hosts > 0) activeCities.push(city.city);
    });

    const gmv = monthlySessions * s.price;
    const appRevenue = gmv * platformTake;
    const costs = s.employees * s.avgSalary + s.fixedOps;
    const net = appRevenue - costs;
    cumulativeCash += net;

    output.push({
      monthLabel: monthLabel(m),
      activeCities: activeCities.join(" + "),
      hostsTotal: monthlyHosts,
      sessions: Math.round(monthlySessions),
      gmv,
      appRevenue,
      costs,
      net,
      cumulativeCash,
    });
  }
  return output;
});

const metrics = computed(() => {
  const totalGmv = rows.value.reduce((acc, r) => acc + r.gmv, 0);
  const totalRevenue = rows.value.reduce((acc, r) => acc + r.appRevenue, 0);
  const totalHostRevenue = totalGmv * 0.7;
  const totalCountryManagerRevenue = totalGmv * 0.04;
  const finalCash = rows.value[rows.value.length - 1]?.cumulativeCash ?? 0;
  const breakEvenRow = rows.value.find((r) => r.cumulativeCash >= 0);
  const positiveFlowRow = rows.value.find((r) => r.net > 0);
  return {
    totalGmv,
    totalRevenue,
    totalHostRevenue,
    totalCountryManagerRevenue,
    finalCash,
    positiveFlow: positiveFlowRow?.monthLabel ?? "Not reached",
    breakEven: breakEvenRow?.monthLabel ?? "Not reached",
  };
});

const euro = (value: number): string =>
  new Intl.NumberFormat("en-GB", { style: "currency", currency: "EUR", maximumFractionDigits: 0 }).format(value);
const euroPrice = (value: number): string =>
  new Intl.NumberFormat("en-GB", {
    style: "currency",
    currency: "EUR",
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(value);

const int = (value: number): string => new Intl.NumberFormat("en-GB").format(value);
const compactEuro = (value: number): string =>
  new Intl.NumberFormat("en-GB", { style: "currency", currency: "EUR", notation: "compact", maximumFractionDigits: 1 }).format(value);
const simulationStartDate = computed(() => {
  const [yearRaw, monthRaw] = scenario.value.initialMonth.split("-");
  const year = Number(yearRaw);
  const month = Number(monthRaw) - 1;
  const safeYear = Number.isFinite(year) ? year : 2026;
  const safeMonth = Number.isFinite(month) ? month : 5;
  return new Date(safeYear, safeMonth, 1).toLocaleDateString("en-GB", { month: "short", year: "numeric" });
});

const cashChart = computed(() => {
  const w = 1000;
  const h = 260;
  const padX = 56;
  const padY = 18;
  if (!rows.value.length) {
    return {
      w,
      h,
      redPoints: "",
      greenPoints: "",
      zeroY: h / 2,
      yTicks: [],
      xTicks: [] as Array<{ x: number; label: string }>,
      breakEvenX: null as number | null,
      breakEvenLabel: "",
    };
  }

  const minCash = Math.min(...rows.value.map((r) => r.cumulativeCash), 0);
  const maxCash = Math.max(...rows.value.map((r) => r.cumulativeCash), 0);
  const ySpan = Math.max(1, maxCash - minCash);
  const usableW = w - padX * 2;
  const usableH = h - padY * 2;
  const points = rows.value.map((r, i) => {
    const x = padX + (i / Math.max(1, rows.value.length - 1)) * usableW;
    const y = padY + (1 - (r.cumulativeCash - minCash) / ySpan) * usableH;
    return { x, y };
  });
  const zeroY = padY + (1 - (0 - minCash) / ySpan) * usableH;

  const breakIdx = rows.value.findIndex((r) => r.cumulativeCash >= 0);
  let breakEvenX: number | null = null;
  let breakEvenLabel = "";
  let redPoints = "";
  let greenPoints = "";

  if (breakIdx === -1) {
    redPoints = points.map((p) => `${p.x.toFixed(2)},${p.y.toFixed(2)}`).join(" ");
  } else if (breakIdx === 0) {
    breakEvenX = points[0].x;
    breakEvenLabel = rows.value[0].monthLabel;
    greenPoints = points.map((p) => `${p.x.toFixed(2)},${p.y.toFixed(2)}`).join(" ");
  } else {
    const prev = points[breakIdx - 1];
    const curr = points[breakIdx];
    const prevCash = rows.value[breakIdx - 1].cumulativeCash;
    const currCash = rows.value[breakIdx].cumulativeCash;
    const t = (0 - prevCash) / Math.max(0.000001, currCash - prevCash);
    const crossX = prev.x + t * (curr.x - prev.x);
    breakEvenX = crossX;
    breakEvenLabel = rows.value[breakIdx].monthLabel;

    const redPath = [...points.slice(0, breakIdx), { x: crossX, y: zeroY }];
    const greenPath = [{ x: crossX, y: zeroY }, ...points.slice(breakIdx)];
    redPoints = redPath.map((p) => `${p.x.toFixed(2)},${p.y.toFixed(2)}`).join(" ");
    greenPoints = greenPath.map((p) => `${p.x.toFixed(2)},${p.y.toFixed(2)}`).join(" ");
  }

  const yTicks = Array.from({ length: 5 }, (_, i) => {
    const v = maxCash - (i / 4) * ySpan;
    const y = padY + (i / 4) * usableH;
    return { y, label: compactEuro(v) };
  });
  const xTickIdx = Array.from(new Set([0, Math.floor((rows.value.length - 1) * 0.33), Math.floor((rows.value.length - 1) * 0.66), rows.value.length - 1]));
  const xTicks = xTickIdx.map((idx) => ({
    x: padX + (idx / Math.max(1, rows.value.length - 1)) * usableW,
    label: rows.value[idx].monthLabel,
  }));

  return { w, h, redPoints, greenPoints, zeroY, yTicks, xTicks, breakEvenX, breakEvenLabel };
});

function normalizePath(path: string): string {
  return path.replace(/\/+$/, "") || "/";
}

function goToPage(target: "portal" | "roi-model") {
  const nextPath = target === "portal" ? "/" : "/roi-model";
  if (normalizePath(window.location.pathname) !== normalizePath(nextPath)) {
    window.history.pushState({}, "", nextPath);
  }
  page.value = target;
}

function syncPageFromPath() {
  page.value = normalizePath(window.location.pathname).endsWith("/roi-model") ? "roi-model" : "portal";
}

onMounted(() => {
  syncPageFromPath();
  window.addEventListener("popstate", syncPageFromPath);
});

onUnmounted(() => {
  window.removeEventListener("popstate", syncPageFromPath);
});
</script>

<template>
  <main class="layout">
    <header class="header">
      <p class="hero-badge">GettaShower investor tool</p>
      <div class="brand-row">
        <img src="/GS-ICON-BLUE.svg" alt="GettaShower icon" class="brand-icon">
        <h1>GettaShower Investor Portal</h1>
      </div>
      <p>
        Interactive forecasting model for Summer 2026 launch and yearly growth. Built on host network expansion,
        session capacity per host, and operating economics.
      </p>
      <nav class="nav">
        <button :class="{ active: page === 'portal' }" @click="goToPage('portal')">Investor portal</button>
        <button :class="{ active: page === 'roi-model' }" @click="goToPage('roi-model')">ROI model</button>
        <a class="home-link-btn" href="https://www.gettashower.com/" target="_blank" rel="noopener noreferrer">GettaShower</a>
      </nav>
      <nav v-if="page === 'roi-model'" class="subnav">
        <button :class="{ active: view === 'simulator' }" @click="view = 'simulator'">Simulator</button>
        <button :class="{ active: view === 'math' }" @click="view = 'math'">Mathematical model</button>
        <button :class="{ active: view === 'reference' }" @click="view = 'reference'">Reference data</button>
      </nav>
    </header>

    <section v-if="page === 'portal'" class="reference portal-page">
      <div class="portal-hero-clean">
        <div class="portal-copy">
          <p class="hero-badge">Investor access</p>
          <h2>Built for the next urban mobility layer.</h2>
          <p>
            GettaShower connects high-frequency urban demand with underused shower infrastructure.
            We monetize existing assets and scale city by city with measurable unit economics.
          </p>
          <div class="portal-cta">
            <a class="home-link-btn pitch-btn" href="https://docs.google.com/presentation/d/1itNnG6QkeZWcX8PBBkrjUuJGBWrQDwzXSwM7T9CfkXM/edit?usp=sharing" target="_blank" rel="noopener noreferrer">Latest pitch deck</a>
          </div>
        </div>
      </div>

      <div class="portal-kpi-row">
        <div class="kpi"><span>Initial investment</span><strong class="neg">{{ euro(scenario.initialInvestment) }}</strong></div>
        <div class="kpi"><span>Projected platform revenue</span><strong class="pos">{{ euro(metrics.totalRevenue) }}</strong></div>
        <div class="kpi"><span>Projected host revenue</span><strong class="pos">{{ euro(metrics.totalHostRevenue) }}</strong></div>
        <div class="kpi"><span>Positive monthly cash flow</span><strong :class="{ pos: metrics.positiveFlow !== 'Not reached' }">{{ metrics.positiveFlow }}</strong></div>
        <div class="kpi"><span>Break-even</span><strong :class="{ pos: metrics.breakEven !== 'Not reached' }">{{ metrics.breakEven }}</strong></div>
        <div class="kpi"><span>Average shower price</span><strong>{{ euroPrice(scenario.price) }}</strong></div>
        <div class="kpi"><span>Average showers/day per host</span><strong>{{ int(scenario.showersPerHostDay) }}</strong></div>
        <div class="kpi"><span>Simulated months</span><strong>{{ int(scenario.months) }}</strong></div>
        <div class="kpi"><span>Initial date of simulation</span><strong>{{ simulationStartDate }}</strong></div>
      </div>
      <div class="portal-actions">
        <button class="home-link-btn simulate-btn" type="button" @click="goToPage('roi-model')">
          Simulate business model
        </button>
      </div>

      <h3>Investment thesis</h3>
      <div class="portal-strip">
        <div><strong>Why now:</strong> high density cities + more active people globally + mobile usage + urban mobility + mobile technology widespread.</div>
        <div><strong>Business model:</strong> 70% host / 20% platform / 6% country manager / 4% CAC.</div>
        <div><strong>Execution:</strong> full technical founding team building and shipping in-house.</div>
      </div>

      <h3>Founder team</h3>
      <div class="portal-grid founders-grid">
        <div class="kpi founder-card">
          <div class="founder-head">
            <span class="founder-photo-ring"><img src="/team/juan-pablo.png" alt="Juan Pablo profile photo" class="founder-photo"></span>
          </div>
          <h4 class="founder-name">Juan Pablo Zúñiga Hidalgo</h4>
          <p class="founder-title">CEO</p>
          <ul class="founder-facts">
            <li><strong>University education:</strong> MSc Engineer + MSc Management (Chalmers University of Technology).</li>
            <li><strong>Role:</strong> Leads product strategy, backend architecture, cloud systems, and GTM execution in Southern Europe.</li>
            <li><strong>Key focus:</strong> city rollout operations, partnerships, and full-stack delivery speed.</li>
          </ul>
          <div class="founder-links">
            <a class="inline-link-btn" href="https://www.linkedin.com/in/jpzuniga" target="_blank" rel="noopener noreferrer">LinkedIn</a>
            <a class="inline-link-btn" href="https://www.zunigajp.com/" target="_blank" rel="noopener noreferrer">Personal site</a>
            <a class="inline-link-btn cv-btn" href="/docs/CV-Juan-Pablo-Zuniga-Hidalgo.pdf" download>Download CV</a>
          </div>
        </div>
        <div class="kpi founder-card">
          <div class="founder-head">
            <span class="founder-photo-ring"><img src="/team/isabell.png" alt="Isabell profile photo" class="founder-photo"></span>
          </div>
          <h4 class="founder-name">Isabell Nordmark</h4>
          <p class="founder-title">CTO</p>
          <ul class="founder-facts">
            <li><strong>University education:</strong> MSc Architecture & Urban Design + MSc Engineer in Computer Science (Chalmers University of Technology).</li>
            <li><strong>Role:</strong> Leads mobile product engineering (Flutter/Dart), technical architecture, and design-quality UX delivery.</li>
            <li><strong>Key focus:</strong> rapid product iteration, user trust flows, and platform usability.</li>
          </ul>
          <div class="founder-links">
            <a class="inline-link-btn" href="https://www.linkedin.com/in/i-nordmark/" target="_blank" rel="noopener noreferrer">LinkedIn</a>
            <a class="inline-link-btn cv-btn" href="/docs/Isabell-Nordmark-CV.pdf" download>Download CV</a>
          </div>
        </div>
      </div>

      <h3>Investor materials</h3>
      <div class="materials-wrap">
        <div class="kpi video-embed-card featured-material">
          <div class="material-header">
            <span class="material-title-main">Concept video</span>
          </div>
          <p>4-minute problem-solution narrative for fast investor context.</p>
          <div class="video-wrap">
            <iframe
              src="https://www.youtube.com/embed/MZTcV0F3MI0"
              title="GettaShower concept video"
              loading="lazy"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
              referrerpolicy="strict-origin-when-cross-origin"
              allowfullscreen
            ></iframe>
          </div>
        </div>

        <div class="kpi material-card featured-material">
            <span class="material-title">Team video</span>
            <p>Founder story, execution readiness, and team chemistry.</p>
            <div class="video-wrap">
              <iframe
                src="https://drive.google.com/file/d/1vz5_miemGXnTkk7ZMMqLidKwsvTDAGvX/preview"
                title="GettaShower team video"
                loading="lazy"
                allow="autoplay; encrypted-media"
                referrerpolicy="strict-origin-when-cross-origin"
                allowfullscreen
              ></iframe>
            </div>
          </div>

      </div>
    </section>

    <section v-if="page === 'roi-model' && view === 'simulator'" class="simulator">
      <aside class="panel">
        <h2>Editable assumptions</h2>
        <div v-if="runError" class="error">{{ runError }}</div>
        <div class="fields">
          <label>
            <span class="label-top">Price per shower (EUR) <span class="info-dot" data-tooltip="Average customer price paid per shower session. Higher values increase GMV and platform revenue directly.">i</span></span>
            <input v-model.number="scenarioDraft.price" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Platform take (%) <span class="info-dot" data-tooltip="Percentage of GMV captured by GettaShower as revenue. Example: 20% means 0.20 x GMV goes to the platform.">i</span></span>
            <input v-model.number="scenarioDraft.platformTakePct" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Host share (%) <span class="info-dot" data-tooltip="Percentage paid to host partners (gyms, hotels, hostels, etc.). Together with other shares, all splits must sum to 100%.">i</span></span>
            <input v-model.number="scenarioDraft.hostTakePct" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Country manager (%) <span class="info-dot" data-tooltip="Share allocated to local country management/distribution efforts. Part of per-session revenue split.">i</span></span>
            <input v-model.number="scenarioDraft.cmPct" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">CAC (%) <span class="info-dot" data-tooltip="Customer acquisition cost share allocated per session. This field is part of the revenue split and must be included in the 100% total.">i</span></span>
            <input v-model.number="scenarioDraft.adsPct" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Host cap used (%) <span class="info-dot" data-tooltip="Percent of estimated city facilities considered realistically capturable as hosts. This dynamically updates city host cap values and model capacity.">i</span></span>
            <input v-model.number="scenarioDraft.hostCapPct" type="number" step="0.1">
          </label>
          <p class="section-label">Host ramp milestones (% of host cap)</p>
          <button class="inline-link-btn" type="button" @click="view = 'reference'">See city caps in Reference data</button>
          <label>
            <span class="label-top">Month 0 (%) <span class="info-dot" data-tooltip="At city launch month, active hosts start at this % of that city's host cap. Host capacity note: City host caps are constrained by market availability and are currently calculated as 20% of estimated city facilities. See city caps in Reference data.">i</span></span>
            <input v-model.number="scenarioDraft.rampPctMonth0" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Month 8 (%) <span class="info-dot" data-tooltip="After 8 months from city launch, active hosts reach this % of city host cap. Host capacity note: City host caps are constrained by market availability and are currently calculated as 20% of estimated city facilities. See city caps in Reference data.">i</span></span>
            <input v-model.number="scenarioDraft.rampPctMonth8" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Month 16 (%) <span class="info-dot" data-tooltip="After 16 months from city launch, active hosts reach this % of city host cap. Host capacity note: City host caps are constrained by market availability and are currently calculated as 20% of estimated city facilities. See city caps in Reference data.">i</span></span>
            <input v-model.number="scenarioDraft.rampPctMonth16" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Month 24+ (%) <span class="info-dot" data-tooltip="Long-term ceiling after 24+ months from city launch. Host capacity note: City host caps are constrained by market availability and are currently calculated as 20% of estimated city facilities. See city caps in Reference data.">i</span></span>
            <input v-model.number="scenarioDraft.rampPctMonth24" type="number" step="0.1">
          </label>
          <label>
            <span class="label-top">Showers per host per day (base) <span class="info-dot" data-tooltip="Average daily shower sessions delivered by each host. Sessions are computed directly from hosts x showers/day x average month length.">i</span></span>
            <input v-model.number="scenarioDraft.showersPerHostDay" type="number" step="1">
          </label>
          <label>
            <span class="label-top">Number of employees <span class="info-dot" data-tooltip="Core team size used to compute monthly payroll cost.">i</span></span>
            <input v-model.number="scenarioDraft.employees" type="number" step="1">
          </label>
          <label>
            <span class="label-top">Average monthly salary (EUR) <span class="info-dot" data-tooltip="Average gross monthly salary per employee. Monthly payroll = employees x average salary.">i</span></span>
            <input v-model.number="scenarioDraft.avgSalary" type="number" step="50">
          </label>
          <label>
            <span class="label-top">Monthly fixed operating costs (EUR) <span class="info-dot" data-tooltip="Recurring non-salary operating expenses (software, legal, operations, admin, etc.).">i</span></span>
            <input v-model.number="scenarioDraft.fixedOps" type="number" step="100">
          </label>
          <label>
            <span class="label-top">Initial investment (EUR) <span class="info-dot" data-tooltip="Upfront capital available at start. Cumulative cash starts at negative this value to measure payback/break-even.">i</span></span>
            <input v-model.number="scenarioDraft.initialInvestment" type="number" step="1000">
          </label>
          <label>
            <span class="label-top">Initial date (month/year) <span class="info-dot" data-tooltip="Simulation starting month used for monthly timeline labels and investor portal KPI card.">i</span></span>
            <input v-model="scenarioDraft.initialMonth" type="month">
          </label>
          <label>
            <span class="label-top">Months simulated <span class="info-dot" data-tooltip="Time horizon for projection. Longer horizons help evaluate break-even and long-term scale.">i</span></span>
            <input v-model.number="scenarioDraft.months" type="number" step="1">
          </label>
        </div>
        <button class="run-btn" :class="{ 'run-btn-success': justRan }" :disabled="Boolean(runError) || !hasPendingChanges" @click="runModel">
          {{ hasPendingChanges ? "Run changes" : "Up to date" }}
        </button>
        <p class="run-status" :class="{ pending: hasPendingChanges }">
          <span v-if="hasPendingChanges">You have pending changes. Click Run to update results.</span>
          <span v-else-if="lastRunAt">Model updated at {{ lastRunAt }}.</span>
          <span v-else>No changes applied yet in this session.</span>
        </p>
      </aside>

      <article class="results">
        <h2>Business model summary</h2>
        <p>
          The model uses a simple operational logic. Each month:
          <strong> Sessions = Hosts x Showers/Host/Day x Average Month Length</strong>.
          This keeps sessions directly proportional to host network size.
        </p>

        <div class="formula">
          <p><strong>Monthly city sessions:</strong> hosts x showers per host per day x {{ AVG_MONTH_LENGTH_DAYS }} days</p>
          <p><strong>Total network sessions:</strong> sum of monthly city sessions</p>
          <p><strong>App revenue:</strong> sessions x price x platform take rate</p>
          <p><strong>Cumulative cash:</strong> previous cash + app revenue - monthly costs</p>
        </div>

        <div class="kpis">
          <div class="kpi"><span>Total transaction value</span><strong>{{ euro(metrics.totalGmv) }}</strong></div>
          <div class="kpi"><span>Platform revenue</span><strong>{{ euro(metrics.totalRevenue) }}</strong></div>
          <div class="kpi"><span>Total Host revenue (70%)</span><strong>{{ euro(metrics.totalHostRevenue) }}</strong></div>
          <div class="kpi"><span>Total Country Manager revenue (4%)</span><strong>{{ euro(metrics.totalCountryManagerRevenue) }}</strong></div>
          <div class="kpi"><span>Final cash position</span><strong :class="{ neg: metrics.finalCash < 0, pos: metrics.finalCash >= 0 }">{{ euro(metrics.finalCash) }}</strong></div>
          <div class="kpi"><span>Positive monthly cash flow</span><strong :class="{ pos: metrics.positiveFlow !== 'Not reached' }">{{ metrics.positiveFlow }}</strong></div>
          <div class="kpi"><span>Break-even</span><strong :class="{ pos: metrics.breakEven !== 'Not reached' }">{{ metrics.breakEven }}</strong></div>
        </div>

        <h3>Monthly projection</h3>
        <div class="table-wrap">
          <table>
            <thead>
              <tr>
                <th>Month</th>
                <th>Active cities</th>
                <th>Hosts</th>
                <th>Sessions</th>
                <th>GMV</th>
                <th>App revenue</th>
                <th>Costs</th>
                <th>Net</th>
                <th>Cash</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in rows" :key="r.monthLabel + r.hostsTotal">
                <td>{{ r.monthLabel }}</td>
                <td>{{ r.activeCities }}</td>
                <td>{{ int(r.hostsTotal) }}</td>
                <td>{{ int(r.sessions) }}</td>
                <td>{{ euro(r.gmv) }}</td>
                <td>{{ euro(r.appRevenue) }}</td>
                <td>{{ euro(r.costs) }}</td>
                <td>{{ euro(r.net) }}</td>
                <td :class="{ neg: r.cumulativeCash < 0, pos: r.cumulativeCash >= 0 }">{{ euro(r.cumulativeCash) }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <h3>Cash vs month</h3>
        <p class="chart-note">Red line: before break-even. Green line: after break-even. Dashed line: zero-cash threshold.</p>
        <div class="chart-wrap">
          <svg :viewBox="`0 0 ${cashChart.w} ${cashChart.h}`" preserveAspectRatio="none" aria-label="Cash versus month chart">
            <g v-for="tick in cashChart.yTicks" :key="`y-${tick.y}`">
              <line x1="56" :y1="tick.y" x2="944" :y2="tick.y" class="grid-line" />
              <text x="6" :y="tick.y + 4" class="axis-text">{{ tick.label }}</text>
            </g>
            <line x1="56" :y1="cashChart.zeroY" x2="944" :y2="cashChart.zeroY" class="zero-line" />
            <line
              v-if="cashChart.breakEvenX !== null"
              :x1="cashChart.breakEvenX"
              y1="18"
              :x2="cashChart.breakEvenX"
              y2="238"
              class="break-even-vline"
            />
            <polyline :points="cashChart.redPoints" class="cash-line-neg" />
            <polyline :points="cashChart.greenPoints" class="cash-line" />
            <g v-for="tick in cashChart.xTicks" :key="`x-${tick.x}`">
              <line :x1="tick.x" y1="238" :x2="tick.x" y2="242" class="x-tick" />
              <text :x="tick.x" y="254" text-anchor="middle" class="axis-text">{{ tick.label }}</text>
            </g>
            <text
              v-if="cashChart.breakEvenX !== null"
              :x="cashChart.breakEvenX"
              y="14"
              text-anchor="middle"
              class="axis-text break-even-label"
            >
              Break-even: {{ cashChart.breakEvenLabel }}
            </text>
          </svg>
        </div>
      </article>
    </section>

    <section v-else-if="page === 'roi-model' && view === 'math'" class="reference">
      <h2>Mathematical model</h2>
      <p>
        This section explains the model step by step, from host rollout to break-even. It is intentionally explicit so that
        investors can audit assumptions and follow each equation.
      </p>
      <p>
        <strong>Plain-language explanation:</strong> each month, the model first estimates how many hosts are active in each city.
        Then it assumes each host delivers an average number of showers per day, multiplied by an average month length (30.4 days).
        That gives monthly sessions. Sessions multiplied by price gives GMV, and the platform take gives GettaShower revenue.
        From that revenue, we subtract monthly team and fixed operating costs to get monthly net cash flow. Finally, monthly net
        cash flow is accumulated over time (starting from initial investment) to compute runway and break-even.
      </p>

      <h3>Step-by-step for non-technical readers</h3>
      <p><strong>Step A:</strong> Count active hosts each month, city by city, following the rollout plan.</p>
      <p><strong>Step B:</strong> Convert hosts into sessions using host productivity (showers/day) and 30.4 days/month.</p>
      <p><strong>Step C:</strong> Convert sessions into money: sessions x price = GMV, then GMV x platform take = app revenue.</p>
      <p><strong>Step D:</strong> Subtract costs (salaries + fixed ops) to get monthly profit/loss.</p>
      <p><strong>Step E:</strong> Add each month to cumulative cash and identify when it first becomes positive (break-even).</p>

      <h3>Step 1: Input variables</h3>
      <table>
        <thead>
          <tr>
            <th>Symbol</th>
            <th>Meaning</th>
            <th>From UI / data</th>
          </tr>
        </thead>
        <tbody>
          <tr><td>H_c,m</td><td>Hosts in city c in month m</td><td>City rollout plan</td></tr>
          <tr><td>q</td><td>Showers per host per day</td><td>UI parameter</td></tr>
          <tr><td>D_avg</td><td>Average month length ({{ AVG_MONTH_LENGTH_DAYS }} days)</td><td>Model constant</td></tr>
          <tr><td>p</td><td>Price per shower</td><td>UI parameter</td></tr>
          <tr><td>r</td><td>Platform take rate</td><td>UI parameter</td></tr>
          <tr><td>C_m</td><td>Monthly operating costs</td><td>Salaries + fixed costs</td></tr>
          <tr><td>I_0</td><td>Initial investment</td><td>UI parameter</td></tr>
        </tbody>
      </table>

      <h3>Current default values used</h3>
      <p><strong>Price per shower:</strong> {{ euro(defaultScenario.price) }}</p>
      <p><strong>Revenue split:</strong> Platform {{ defaultScenario.platformTakePct }}% / Host {{ defaultScenario.hostTakePct }}% / Country manager {{ defaultScenario.cmPct }}% / Advertising {{ defaultScenario.adsPct }}%</p>
      <p><strong>Host cap used:</strong> {{ defaultScenario.hostCapPct }}%</p>
      <p><strong>Showers per host per day:</strong> {{ defaultScenario.showersPerHostDay }}</p>
      <p><strong>Employees:</strong> {{ defaultScenario.employees }}</p>
      <p><strong>Average monthly salary:</strong> {{ euro(defaultScenario.avgSalary) }}</p>
      <p><strong>Monthly fixed operating costs:</strong> {{ euro(defaultScenario.fixedOps) }}</p>
      <p><strong>Initial investment:</strong> {{ euro(defaultScenario.initialInvestment) }}</p>
      <p><strong>Months simulated:</strong> {{ defaultScenario.months }}</p>

      <h3>Step 2: City host rollout</h3>
      <p>
        Hosts are modeled city by city with a progressive rollout. Launch order is:
        <code>Barcelona -&gt; Rome -&gt; Madrid -&gt; Milan -&gt; Paris -&gt; Munich -&gt; Athens</code>.
        A new city is activated every 4 months.
      </p>
      <p>
        A realistic host ceiling is enforced from reference data:
        <code>HostCap_city = {{ scenario.hostCapPct }}% x estimated_total_facilities_city</code>.
      </p>
      <p>
        After a city launches, host activation follows this path:
        <code>{{ scenario.rampPctMonth0 }}% at month 0</code>,
        <code>{{ scenario.rampPctMonth8 }}% at month 8</code>,
        <code>{{ scenario.rampPctMonth16 }}% at month 16</code>,
        and <code>{{ scenario.rampPctMonth24 }}% at month 24</code>.
      </p>

      <h3>Step 3: Sessions per city</h3>
      <p><strong>Monthly sessions in each city:</strong></p>
      <p><code>Sessions_c,m = H_c,m * q * D_avg</code></p>

      <h3>Step 4: Network-level sessions</h3>
      <p><strong>Total sessions across all cities:</strong></p>
      <p><code>Sessions_m = sum over cities (Sessions_c,m)</code></p>

      <h3>Step 5: Revenue and costs</h3>
      <p><strong>Gross Merchandise Value (GMV):</strong></p>
      <p><code>GMV_m = Sessions_m * p</code></p>
      <p><strong>Platform revenue:</strong></p>
      <p><code>Revenue_m = GMV_m * r</code></p>
      <p><strong>Monthly costs:</strong></p>
      <p><code>C_m = (employees * avg_salary) + fixed_ops</code></p>
      <p><strong>Monthly net cash flow:</strong></p>
      <p><code>Net_m = Revenue_m - C_m</code></p>

      <h3>Step 6: Cumulative cash and break-even</h3>
      <p><strong>Initial cash position:</strong> <code>Cash_-1 = -I_0</code></p>
      <p><strong>Cumulative cash:</strong></p>
      <p><code>Cash_m = Cash_(m-1) + Net_m</code></p>
      <p><strong>Break-even month:</strong> first month where <code>Cash_m >= 0</code>.</p>

      <h3>Worked mini example (one month)</h3>
      <p>
        Suppose in month m: <code>Sessions_m = 20,000</code>, <code>p = 10 EUR</code>, <code>r = 20%</code>,
        <code>employees = 2</code>, <code>avg_salary = 6,000</code>, <code>fixed_ops = 1,500</code>.
      </p>
      <p><code>GMV_m = 20,000 * 10 = 200,000 EUR</code></p>
      <p><code>Revenue_m = 200,000 * 0.20 = 40,000 EUR</code></p>
      <p><code>C_m = 2 * 6,000 + 1,500 = 13,500 EUR</code></p>
      <p><code>Net_m = 40,000 - 13,500 = 26,500 EUR</code></p>
      <p>
        Cumulative cash then updates as: <code>Cash_m = Cash_(m-1) + 26,500</code>.
      </p>
    </section>

    <section v-else-if="page === 'roi-model'" class="reference">
      <h2>Reference data used</h2>
      <p>
        This page shows the CSV input used by the model. The
        <strong> average_monthly_tourists </strong> column is the base demand signal for each city.
      </p>
      <pre class="csv">{{ csvRaw }}</pre>

      <h3>Parsed table used for calculation</h3>
      <table>
        <thead>
          <tr>
            <th>City</th>
            <th>Population</th>
            <th>Average tourists / month</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="city in cityData" :key="city.city">
            <td>{{ city.city }}</td>
            <td>{{ int(city.population) }}</td>
            <td>{{ int(city.averageMonthlyTourists) }}</td>
          </tr>
        </tbody>
      </table>
      <h3 style="margin-top: 1rem;">Host reference data (Gemini estimate)</h3>
      <p>
        Estimated total facilities per city (gyms, hostels, hotels, spas, sports centers). Model host cap uses {{ scenario.hostCapPct }}% of this.
      </p>
      <p>
        Quick formula: <code>HostCap_city = estimated_total_facilities_city x {{ (scenario.hostCapPct / 100).toFixed(2) }}</code>
      </p>
      <pre class="csv">{{ hostFacilitiesRaw }}</pre>
      <table>
        <thead>
          <tr>
            <th>City</th>
            <th>Estimated total facilities</th>
            <th>Host cap used ({{ scenario.hostCapPct }}%)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="entry in hostFacilitiesData" :key="entry.city">
            <td>{{ entry.city }}</td>
            <td>{{ int(entry.estimatedTotalFacilities) }}</td>
            <td>{{ int(Math.round(entry.estimatedTotalFacilities * (scenario.hostCapPct / 100))) }}</td>
          </tr>
        </tbody>
      </table>
      <p style="margin-top: 0.7rem;">
        Rollout assumptions in this version: Barcelona, then Rome, then Madrid, then Milan, then Paris, then Munich, and finally Athens.
        One new city every 4 months, and each city ramps according to the variable milestones set in the simulator.
      </p>
      <p>
        Recommended first 4 priority cities for business fit: <strong>Barcelona, Rome, Madrid, Milan</strong>
        (strong tourism flow + urban density + high practical need for on-the-go showers).
      </p>
    </section>
  </main>
</template>
