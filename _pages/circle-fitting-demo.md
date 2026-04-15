```html
---
layout: single
title: "Interactive Circle Fitting Demo"
permalink: /circle-fitting-demo/
author_profile: false
classes: wide
---

<div style="width:100%; margin-bottom:32px;">
  <video autoplay muted loop playsinline 
         style="width:100%; max-width:1200px; display:block; margin:auto; border-radius:12px;">
    <source src="/assets/videos/demo.mp4" type="video/mp4">
  </video>
</div>

<style>
  .wrapper {
    max-width: 100% !important;
  }

  .page {
    max-width: 100% !important;
    padding-left: 32px !important;
    padding-right: 32px !important;
  }

  .page__content {
    max-width: 100% !important;
  }

  #main {
    max-width: 100% !important;
  }

  .cf-container {
    width: 100%;
    max-width: 1600px;
    margin: 0 auto;
  }

  .cf-grid {
    display: grid;
    grid-template-columns: 320px minmax(0, 1fr);
    gap: 24px;
    align-items: start;
  }

  .cf-panel {
    border: 1px solid #ddd;
    border-radius: 12px;
    padding: 16px;
    position: sticky;
    top: 20px;
    background: #fff;
  }

  .cf-group {
    margin-bottom: 16px;
  }

  .cf-group label {
    display: block;
    font-weight: 600;
    margin-bottom: 6px;
  }

  .cf-group input[type="range"],
  .cf-group button {
    width: 100%;
  }

  .cf-value {
    font-size: 0.95rem;
    color: #555;
  }

  .cf-note {
    border-left: 4px solid #888;
    background: #f7f7f7;
    padding: 12px 14px;
    margin-bottom: 18px;
  }

  .cf-box {
    border: 1px solid #ddd;
    border-radius: 12px;
    padding: 14px;
    background: #fafafa;
    margin-top: 16px;
  }

  .cf-plot {
    margin-bottom: 24px;
  }

  .cf-status {
    font-size: 0.95rem;
    color: #444;
    margin-top: 8px;
    min-height: 24px;
  }

  .cf-small {
    font-size: 0.9rem;
    color: #666;
  }

  @media (max-width: 980px) {
    .page {
      padding-left: 16px !important;
      padding-right: 16px !important;
    }

    .cf-grid {
      grid-template-columns: 1fr;
    }

    .cf-panel {
      position: static;
    }
  }
</style>

<div class="cf-container">
  <div class="cf-note">
    This page shows only the proposed local z-score outlier removal method.
    Blue points are retained points. Red points are detected outliers.
  </div>

  <div class="cf-grid">
    <div class="cf-panel">
      <div class="cf-group">
        <label for="sigma">Sigma</label>
        <input type="range" id="sigma" min="0" max="5" step="0.1" value="0.8">
        <div class="cf-value"><span id="sigma_val">0.8</span></div>
      </div>

      <div class="cf-group">
        <label for="n_points">n_points</label>
        <input type="range" id="n_points" min="100" max="3000" step="50" value="1000">
        <div class="cf-value"><span id="n_points_val">1000</span></div>
      </div>

      <div class="cf-group">
        <label for="a">a</label>
        <input type="range" id="a" min="100" max="1000" step="5" value="675">
        <div class="cf-value"><span id="a_val">675</span></div>
      </div>

      <div class="cf-group">
        <label for="b">b</label>
        <input type="range" id="b" min="100" max="1000" step="5" value="685">
        <div class="cf-value"><span id="b_val">685</span></div>
      </div>

      <div class="cf-group">
        <label for="cluster_ratio">Cluster Outlier Ratio</label>
        <input type="range" id="cluster_ratio" min="0" max="0.20" step="0.005" value="0.02">
        <div class="cf-value"><span id="cluster_ratio_val">0.02</span></div>
      </div>

      <div class="cf-group">
        <label for="near_ratio">Near-Ellipse Outlier Ratio</label>
        <input type="range" id="near_ratio" min="0" max="0.20" step="0.005" value="0.02">
        <div class="cf-value"><span id="near_ratio_val">0.02</span></div>
      </div>

      <div class="cf-group cf-row-btns">
        <button id="run_btn">Run Demo</button>
      </div>

      <div class="cf-small">
        Current settings:<br>
        cluster_outliers = int(n_points × cluster_ratio)<br>
        near_ellipse_outliers = int(n_points × near_ratio)<br>
        random_outliers = 0<br>
        random_seed = 42
      </div>

      <div class="cf-status" id="status_box">Ready.</div>

      <div class="cf-box" id="result_box">
        Results will appear here.
      </div>
    </div>

    <div>
      <div class="cf-plot">
        <div id="plot_scatter" style="width:100%;height:760px;"></div>
      </div>

      <div class="cf-plot">
        <div id="plot_rtheta" style="width:100%;height:420px;"></div>
      </div>
    </div>
  </div>
</div>

<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

<script>
function mulberry32(a) {
  return function() {
    let t = a += 0x6D2B79F5;
    t = Math.imul(t ^ (t >>> 15), t | 1);
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

function randn(rng) {
  let u = 0, v = 0;
  while (u === 0) u = rng();
  while (v === 0) v = rng();
  return Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);
}

function mean(arr) {
  return arr.reduce((a, b) => a + b, 0) / arr.length;
}

function median(arr) {
  const s = [...arr].sort((a, b) => a - b);
  const n = s.length;
  return n % 2 ? s[(n - 1) / 2] : 0.5 * (s[n / 2 - 1] + s[n / 2]);
}

function std(arr) {
  const m = mean(arr);
  return Math.sqrt(mean(arr.map(v => (v - m) * (v - m))));
}

function linspace(start, end, n, endpoint = true) {
  const arr = [];
  if (n <= 1) return [start];
  const step = endpoint ? (end - start) / (n - 1) : (end - start) / n;
  for (let i = 0; i < n; i++) {
    arr.push(start + i * step);
  }
  return arr;
}

function shuffleTogether(arrs, rng) {
  const n = arrs[0].length;
  for (let i = n - 1; i > 0; i--) {
    const j = Math.floor(rng() * (i + 1));
    for (const arr of arrs) {
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
}

function generateSyntheticEllipse({
  xc = 500,
  yc = 500,
  a = 675,
  b = 685,
  n_points = 1000,
  sigma = 0.8,
  cluster_outliers = 20,
  near_ellipse_outliers = 20,
  random_outliers = 0,
  random_seed = 42
}) {
  const rng = mulberry32(random_seed);

  const theta = linspace(0, 2 * Math.PI, n_points, false);
  const x = theta.map(t => xc + a * Math.cos(t));
  const y = theta.map(t => yc + b * Math.sin(t));

  const x_in = x.map(v => v + sigma * randn(rng));
  const y_in = y.map(v => v + sigma * randn(rng));

  const x_out_cluster = [];
  const y_out_cluster = [];
  for (let i = 0; i < cluster_outliers; i++) {
    x_out_cluster.push((xc + a + 5) + 1 * randn(rng));
    y_out_cluster.push(yc + 1 * randn(rng));
  }

  const x_out_near = [];
  const y_out_near = [];
  for (let i = 0; i < near_ellipse_outliers; i++) {
    const t = 2 * Math.PI * rng();
    const scale = 1 + 0.01 * randn(rng);
    x_out_near.push(xc + scale * a * Math.cos(t));
    y_out_near.push(yc + scale * b * Math.sin(t));
  }

  const x_out_random = [];
  const y_out_random = [];
  const span = 1.5 * Math.max(a, b) * 2;
  for (let i = 0; i < random_outliers; i++) {
    x_out_random.push((xc - span) + (2 * span) * rng());
    y_out_random.push((yc - span) + (2 * span) * rng());
  }

  const X = [...x_in, ...x_out_cluster, ...x_out_near, ...x_out_random];
  const Y = [...y_in, ...y_out_cluster, ...y_out_near, ...y_out_random];
  const labels = [
    ...Array(x_in.length).fill(0),
    ...Array(x_out_cluster.length).fill(1),
    ...Array(x_out_near.length).fill(2),
    ...Array(x_out_random.length).fill(3)
  ];

  shuffleTogether([X, Y, labels], rng);

  return {
    x_in, y_in,
    x_out_cluster, y_out_cluster,
    x_out_near, y_out_near,
    x_out_random, y_out_random,
    X, Y, labels
  };
}

function removeOutliersLocalZScoreProposed(x, y, threshold = 3, window_size = 60, std_window = 60) {
  const xc = mean(x);
  const yc = mean(y);

  const theta = [];
  const r = [];
  for (let i = 0; i < x.length; i++) {
    theta.push(Math.atan2(y[i] - yc, x[i] - xc));
    r.push(Math.hypot(x[i] - xc, y[i] - yc));
  }

  const idx = Array.from(theta.keys()).sort((a, b) => theta[a] - theta[b]);
  const theta_sorted = idx.map(i => theta[i]);
  const r_sorted = idx.map(i => r[i]);
  const x_sorted = idx.map(i => x[i]);
  const y_sorted = idx.map(i => y[i]);

  const std_list = [];
  const stride = 20;
  for (let i = 0; i <= r_sorted.length - std_window; i += stride) {
    std_list.push(std(r_sorted.slice(i, i + std_window)));
  }

  const global_std = (std_list.length === 0 ? std(r_sorted) : median(std_list)) + 1e-12;

  const n = r_sorted.length;
  const mask = Array(n).fill(true);

  if (n < window_size) {
    const mean_r = mean(r_sorted);
    const outliers = r_sorted.map(v => Math.abs(v - mean_r) > threshold * global_std);
    for (let i = 0; i < n; i++) {
      mask[i] = !outliers[i];
    }
  } else {
    for (let i = 0; i <= n - window_size; i++) {
      const window = r_sorted.slice(i, i + window_size);
      const mean_r = mean(window);
      const outliers = window.map(v => Math.abs(v - mean_r) > threshold * global_std);

      for (let j = 0; j < window_size; j++) {
        mask[i + j] = mask[i + j] && !outliers[j];
      }
    }
  }

  const x_kept = [];
  const y_kept = [];
  const x_removed = [];
  const y_removed = [];
  const theta_removed = [];
  const r_removed = [];

  for (let i = 0; i < n; i++) {
    if (mask[i]) {
      x_kept.push(x_sorted[i]);
      y_kept.push(y_sorted[i]);
    } else {
      x_removed.push(x_sorted[i]);
      y_removed.push(y_sorted[i]);
      theta_removed.push(theta_sorted[i]);
      r_removed.push(r_sorted[i]);
    }
  }

  return {
    x_kept,
    y_kept,
    x_removed,
    y_removed,
    theta_sorted,
    r_sorted,
    theta_removed,
    r_removed
  };
}

function setStatus(msg) {
  document.getElementById('status_box').textContent = msg;
}

function updateSliderValues() {
  document.getElementById('sigma_val').textContent = document.getElementById('sigma').value;
  document.getElementById('n_points_val').textContent = document.getElementById('n_points').value;
  document.getElementById('a_val').textContent = document.getElementById('a').value;
  document.getElementById('b_val').textContent = document.getElementById('b').value;
  document.getElementById('cluster_ratio_val').textContent = document.getElementById('cluster_ratio').value;
  document.getElementById('near_ratio_val').textContent = document.getElementById('near_ratio').value;
}

function currentParams() {
  return {
    sigma: parseFloat(document.getElementById('sigma').value),
    n_points: parseInt(document.getElementById('n_points').value, 10),
    a: parseFloat(document.getElementById('a').value),
    b: parseFloat(document.getElementById('b').value),
    cluster_ratio: parseFloat(document.getElementById('cluster_ratio').value),
    near_ratio: parseFloat(document.getElementById('near_ratio').value)
  };
}

function renderScatter(result) {
  const traces = [
    {
      x: result.x_kept,
      y: result.y_kept,
      mode: 'markers',
      type: 'scatter',
      name: 'Points',
      marker: { size: 4, color: '#1f77b4' }
    },
    {
      x: result.x_removed,
      y: result.y_removed,
      mode: 'markers',
      type: 'scatter',
      name: 'Outliers',
      marker: { size: 6, color: '#d62728' }
    }
  ];

  Plotly.newPlot('plot_scatter', traces, {
    title: 'Detected Outliers',
    xaxis: { title: 'x', scaleanchor: 'y' },
    yaxis: { title: 'y' },
    legend: { orientation: 'h' }
  }, { responsive: true });
}

function renderRTheta(result) {
  const traces = [
    {
      x: result.theta_sorted,
      y: result.r_sorted,
      mode: 'markers',
      type: 'scatter',
      name: 'Points',
      marker: { size: 4, color: '#1f77b4' }
    },
    {
      x: result.theta_removed,
      y: result.r_removed,
      mode: 'markers',
      type: 'scatter',
      name: 'Outliers',
      marker: { size: 6, color: '#d62728' }
    }
  ];

  Plotly.newPlot('plot_rtheta', traces, {
    title: 'r-theta Plot',
    xaxis: { title: 'theta (radian)' },
    yaxis: { title: 'r' },
    legend: { orientation: 'h' }
  }, { responsive: true });
}

function renderResultsBox(summary) {
  document.getElementById('result_box').innerHTML = `
    <strong>Proposed Local Z-Score</strong><br><br>
    <strong>Total points:</strong> ${summary.total}<br>
    <strong>Detected outliers:</strong> ${summary.removed}<br>
    <strong>Remaining points:</strong> ${summary.remaining}
  `;
}

function runDemo() {
  try {
    setStatus("Running...");

    const params = currentParams();
    const xc = 500;
    const yc = 500;
    const cluster_outliers = Math.floor(params.n_points * params.cluster_ratio);
    const near_ellipse_outliers = Math.floor(params.n_points * params.near_ratio);

    const data = generateSyntheticEllipse({
      xc,
      yc,
      a: params.a,
      b: params.b,
      n_points: params.n_points,
      sigma: params.sigma,
      cluster_outliers,
      near_ellipse_outliers,
      random_outliers: 0,
      random_seed: 42
    });

    const result = removeOutliersLocalZScoreProposed(data.X, data.Y, 3, 60, 60);

    renderScatter(result);
    renderRTheta(result);
    renderResultsBox({
      total: data.X.length,
      removed: result.x_removed.length,
      remaining: result.x_kept.length
    });

    setStatus("Completed.");
  } catch (err) {
    console.error(err);
    setStatus("Error: " + err.message);
  }
}

document.querySelectorAll('input[type="range"]').forEach(el => {
  el.addEventListener('input', updateSliderValues);
});

document.getElementById('run_btn').addEventListener('click', runDemo);

updateSliderValues();
runDemo();
</script>
```
