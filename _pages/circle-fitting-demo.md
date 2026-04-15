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

  .cf-table-wrap {
    overflow-x: auto;
  }

  .cf-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.92rem;
  }

  .cf-table th,
  .cf-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
    white-space: nowrap;
  }

  .cf-table th {
    background: #f2f2f2;
  }

  .cf-row-btns {
    display: flex;
    gap: 8px;
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
    This page runs directly in JavaScript for speed.
    Adjustable variables are <code>sigma</code>, <code>n_points</code>, <code>a</code>, <code>b</code>,
    <code>cluster_ratio</code>, and <code>near_ratio</code>.
    Only the <code>Proposed Local Z-Score</code> method is used.
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
        <button id="clear_table_btn" type="button">Clear Table</button>
      </div>

      <div class="cf-status" id="status_box">Ready.</div>

    </div>

    <div>
      <div class="cf-plot">
        <div id="plot_scatter" style="width:100%;height:760px;"></div>
      </div>

      <div class="cf-plot">
        <div id="plot_rtheta" style="width:100%;height:420px;"></div>
      </div>

      <div class="cf-box">
        <h3 style="margin-top:0;">Run History</h3>
        <div class="cf-table-wrap">
          <table class="cf-table" id="results_table">
            <thead>
              <tr>
                <th>#</th>
                <th>Sigma</th>
                <th>n_points</th>
                <th>a</th>
                <th>b</th>
                <th>Cluster Ratio</th>
                <th>Near Ratio</th>
                <th>Total</th>
                <th>Removed</th>
                <th>Remaining</th>
                <th>Estimated xc</th>
                <th>Estimated yc</th>
                <th>Estimated r</th>
                <th>Reference r</th>
                <th>Center Err</th>
                <th>Radius Err</th>
              </tr>
            </thead>
            <tbody></tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</div>

<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

<script>
let runCounter = 0;

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

function solve3x3(A, b) {
  const M = A.map((row, i) => [...row, b[i]]);
  for (let i = 0; i < 3; i++) {
    let maxRow = i;
    for (let k = i + 1; k < 3; k++) {
      if (Math.abs(M[k][i]) > Math.abs(M[maxRow][i])) maxRow = k;
    }
    [M[i], M[maxRow]] = [M[maxRow], M[i]];
    const piv = M[i][i];
    if (Math.abs(piv) < 1e-12) throw new Error("Singular matrix");
    for (let j = i; j < 4; j++) M[i][j] /= piv;
    for (let k = 0; k < 3; k++) {
      if (k !== i) {
        const factor = M[k][i];
        for (let j = i; j < 4; j++) M[k][j] -= factor * M[i][j];
      }
    }
  }
  return [M[0][3], M[1][3], M[2][3]];
}

function solve2x2(A, b) {
  const det = A[0][0] * A[1][1] - A[0][1] * A[1][0];
  if (Math.abs(det) < 1e-12) throw new Error("Singular 2x2");
  const x = (b[0] * A[1][1] - b[1] * A[0][1]) / det;
  const y = (A[0][0] * b[1] - A[1][0] * b[0]) / det;
  return [x, y];
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

function computeRadiusFromCenter(x, y, xc, yc) {
  const r = [];
  for (let i = 0; i < x.length; i++) {
    r.push(Math.hypot(x[i] - xc, y[i] - yc));
  }
  return r;
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

  const x_filt = [];
  const y_filt = [];

  for (let i = 0; i < n; i++) {
    if (mask[i]) {
      x_filt.push(x_sorted[i]);
      y_filt.push(y_sorted[i]);
    }
  }

  return { x: x_filt, y: y_filt };
}

function fitTaubin(x, y) {
  const xm = mean(x);
  const ym = mean(y);
  const u = x.map(v => v - xm);
  const v = y.map(vv => vv - ym);

  let Suu = 0, Suv = 0, Svv = 0, Suuu = 0, Suvv = 0, Svvv = 0, Svuu = 0;

  for (let i = 0; i < x.length; i++) {
    Suu += u[i] * u[i];
    Suv += u[i] * v[i];
    Svv += v[i] * v[i];
    Suuu += u[i] * u[i] * u[i];
    Suvv += u[i] * v[i] * v[i];
    Svvv += v[i] * v[i] * v[i];
    Svuu += v[i] * u[i] * u[i];
  }

  const A = [[Suu, Suv], [Suv, Svv]];
  const B = [0.5 * (Suuu + Suvv), 0.5 * (Svvv + Svuu)];
  const [uc, vc] = solve2x2(A, B);

  const xc = xm + uc;
  const yc = ym + vc;
  const r = Math.sqrt(uc * uc + vc * vc + (Suu + Svv) / x.length);
  return [xc, yc, r];
}

function fitGeometricLS(x, y, iterations = 12) {
  let [xc, yc, r] = fitTaubin(x, y);

  for (let it = 0; it < iterations; it++) {
    let J11 = 0, J12 = 0, J13 = 0, J22 = 0, J23 = 0, J33 = x.length;
    let B1 = 0, B2 = 0, B3 = 0;

    for (let i = 0; i < x.length; i++) {
      const dx = xc - x[i];
      const dy = yc - y[i];
      const di = Math.sqrt(dx * dx + dy * dy) + 1e-12;
      const fi = di - r;

      const j1 = dx / di;
      const j2 = dy / di;
      const j3 = -1;

      J11 += j1 * j1;
      J12 += j1 * j2;
      J13 += j1 * j3;
      J22 += j2 * j2;
      J23 += j2 * j3;
      B1 += j1 * fi;
      B2 += j2 * fi;
      B3 += j3 * fi;
    }

    const delta = solve3x3(
      [[J11, J12, J13], [J12, J22, J23], [J13, J23, J33]],
      [-B1, -B2, -B3]
    );

    xc += delta[0];
    yc += delta[1];
    r += delta[2];

    if (Math.abs(delta[0]) + Math.abs(delta[1]) + Math.abs(delta[2]) < 1e-6) break;
  }

  return [xc, yc, r];
}

function plotRThetaArrays(x, y, xc_ref, yc_ref) {
  const data = [];

  for (let i = 0; i < x.length; i++) {
    const th = Math.atan2(y[i] - yc_ref, x[i] - xc_ref);
    const rr = Math.hypot(x[i] - xc_ref, y[i] - yc_ref);
    data.push({ th, rr });
  }

  data.sort((a, b) => a.th - b.th);

  return {
    theta: data.map(v => v.th),
    r: data.map(v => v.rr)
  };
}

function splitDetectedOutliers(allX, allY, filtX, filtY) {
  const removedX = [];
  const removedY = [];
  const keptX = [];
  const keptY = [];
  const used = new Array(filtX.length).fill(false);

  for (let i = 0; i < allX.length; i++) {
    let foundIndex = -1;

    for (let j = 0; j < filtX.length; j++) {
      if (used[j]) continue;

      if (
        Math.abs(allX[i] - filtX[j]) < 1e-9 &&
        Math.abs(allY[i] - filtY[j]) < 1e-9
      ) {
        foundIndex = j;
        break;
      }
    }

    if (foundIndex >= 0) {
      keptX.push(allX[i]);
      keptY.push(allY[i]);
      used[foundIndex] = true;
    } else {
      removedX.push(allX[i]);
      removedY.push(allY[i]);
    }
  }

  return { keptX, keptY, removedX, removedY };
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

function renderScatter(scatter) {
  const traces = [
    {
      x: scatter.x_kept,
      y: scatter.y_kept,
      mode: 'markers',
      type: 'scatter',
      name: 'Points',
      marker: { size: 4, color: '#1f77b4' }
    },
    {
      x: scatter.x_removed,
      y: scatter.y_removed,
      mode: 'markers',
      type: 'scatter',
      name: 'Outliers',
      marker: { size: 7, color: '#d62728' }
    },
    {
      x: scatter.x_circle_fit,
      y: scatter.y_circle_fit,
      mode: 'lines',
      type: 'scatter',
      name: 'Fitted Circle',
      line: { width: 3, color: '#9467bd' }
    },
    {
      x: [scatter.x_est_center],
      y: [scatter.y_est_center],
      mode: 'markers',
      type: 'scatter',
      name: 'Estimated Center',
      marker: { size: 10, symbol: 'square', color: '#ff1493' }
    }
  ];

  Plotly.newPlot('plot_scatter', traces, {
    title: 'Point Distribution and Detected Outliers',
    xaxis: { title: 'x', scaleanchor: 'y' },
    yaxis: { title: 'y' },
    legend: { orientation: 'h' }
  }, { responsive: true });
}

function renderRTheta(rtheta) {
  const traces = [
    {
      x: rtheta.theta_original,
      y: rtheta.r_original,
      mode: 'markers',
      type: 'scatter',
      name: 'Original',
      marker: { size: 4, color: '#1f77b4' }
    },
    {
      x: rtheta.theta_removed,
      y: rtheta.r_removed,
      mode: 'markers',
      type: 'scatter',
      name: 'Detected Outliers',
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


function appendResultRow(data, params) {
  runCounter += 1;
  const tbody = document.querySelector('#results_table tbody');
  const tr = document.createElement('tr');

  tr.innerHTML = `
    <td>${runCounter}</td>
    <td>${params.sigma.toFixed(2)}</td>
    <td>${params.n_points}</td>
    <td>${params.a.toFixed(0)}</td>
    <td>${params.b.toFixed(0)}</td>
    <td>${params.cluster_ratio.toFixed(3)}</td>
    <td>${params.near_ratio.toFixed(3)}</td>
    <td>${data.counts.total}</td>
    <td>${data.counts.removed}</td>
    <td>${data.counts.remaining}</td>
    <td>${data.selected_result.x0.toFixed(4)}</td>
    <td>${data.selected_result.y0.toFixed(4)}</td>
    <td>${data.selected_result.r.toFixed(4)}</td>
    <td>${data.selected_result.r_ref.toFixed(4)}</td>
    <td>${data.selected_result.center_error.toFixed(4)}</td>
    <td>${data.selected_result.radius_error.toFixed(4)}</td>
  `;

  tbody.appendChild(tr);
}

function clearResultTable() {
  document.querySelector('#results_table tbody').innerHTML = '';
  runCounter = 0;
}

function runExperiment(params) {
  const xc = 500;
  const yc = 500;
  const a = params.a;
  const b = params.b;
  const n_points = params.n_points;
  const sigma = params.sigma;

  const cluster_outliers = Math.floor(n_points * params.cluster_ratio);
  const near_ellipse_outliers = Math.floor(n_points * params.near_ratio);
  const random_outliers = 0;
  const random_seed = 42;

  const data = generateSyntheticEllipse({
    xc, yc, a, b, n_points, sigma,
    cluster_outliers, near_ellipse_outliers, random_outliers, random_seed
  });

  const cleanedSelected = removeOutliersLocalZScoreProposed(data.X, data.Y);
  const [x0_sel, y0_sel, r_sel] = fitGeometricLS(cleanedSelected.x, cleanedSelected.y);

  const r_ref = (a + b) / 2.0;
  const center_error_sel = Math.hypot(x0_sel - xc, y0_sel - yc);
  const radius_error_sel = Math.abs(r_sel - r_ref);

  const splitDetected = splitDetectedOutliers(
    data.X,
    data.Y,
    cleanedSelected.x,
    cleanedSelected.y
  );

  const rtOrig = plotRThetaArrays(data.X, data.Y, xc, yc);
  const rtRemoved = splitDetected.removedX.length > 0
    ? plotRThetaArrays(splitDetected.removedX, splitDetected.removedY, xc, yc)
    : { theta: [], r: [] };

  const th = linspace(0, 2 * Math.PI, 400, true);
  const x_circle_fit = th.map(t => x0_sel + r_sel * Math.cos(t));
  const y_circle_fit = th.map(t => y0_sel + r_sel * Math.sin(t));

  return {
    counts: {
      total: data.X.length,
      remaining: cleanedSelected.x.length,
      removed: data.X.length - cleanedSelected.x.length,
      true_outlier_total: data.x_out_cluster.length + data.x_out_near.length + data.x_out_random.length
    },
    scatter: {
      x_kept: splitDetected.keptX,
      y_kept: splitDetected.keptY,
      x_removed: splitDetected.removedX,
      y_removed: splitDetected.removedY,
      x_circle_fit,
      y_circle_fit,
      x_est_center: x0_sel,
      y_est_center: y0_sel
    },
    rtheta: {
      theta_original: rtOrig.theta,
      r_original: rtOrig.r,
      theta_removed: rtRemoved.theta,
      r_removed: rtRemoved.r
    },
    selected_result: {
      x0: x0_sel,
      y0: y0_sel,
      r: r_sel,
      r_ref: r_ref,
      center_error: center_error_sel,
      radius_error: radius_error_sel
    }
  };
}

function runDemo() {
  try {
    setStatus("Running...");
    const params = currentParams();
    const result = runExperiment(params);
    renderScatter(result.scatter);
    renderRTheta(result.rtheta);
    appendResultRow(result, params);
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
document.getElementById('clear_table_btn').addEventListener('click', clearResultTable);

updateSliderValues();
runDemo();
</script>
