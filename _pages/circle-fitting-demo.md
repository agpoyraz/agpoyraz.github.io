---
layout: single
title: "Interactive Circle Fitting Demo"
permalink: /circle-fitting-demo/
author_profile: false
---

<style>
  .page {
    max-width: 100% !important;
  }

  .cf-container {
    width: 100%;
  }

  .cf-grid {
    display: grid;
    grid-template-columns: 340px 1fr;
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

  .cf-small {
    font-size: 0.9rem;
    color: #666;
  }

  .cf-row-btns {
    display: flex;
    gap: 8px;
  }

  @media (max-width: 980px) {
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
    Only the proposed outlier detection method is used.
    Adjustable variables are <code>sigma</code>, <code>n_points</code>, <code>a</code>, <code>b</code>,
    <code>cluster_ratio</code>, <code>near_ratio</code>, <code>threshold</code>,
    <code>window_size</code>, and <code>std_window</code>.
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

      <div class="cf-group">
        <label for="threshold">threshold</label>
        <input type="range" id="threshold" min="0.5" max="10" step="0.1" value="3">
        <div class="cf-value"><span id="threshold_val">3</span></div>
      </div>

      <div class="cf-group">
        <label for="window_size">window_size</label>
        <input type="range" id="window_size" min="5" max="300" step="1" value="60">
        <div class="cf-value"><span id="window_size_val">60</span></div>
      </div>

      <div class="cf-group">
        <label for="std_window">std_window</label>
        <input type="range" id="std_window" min="5" max="300" step="1" value="60">
        <div class="cf-value"><span id="std_window_val">60</span></div>
      </div>

      <div class="cf-group cf-row-btns">
        <button id="run_btn">Run Demo</button>
        <button id="clear_table_btn" type="button">Clear Table</button>
      </div>

      <div class="cf-small">
        Method: Proposed Local Z-Score<br>
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
                <th>threshold</th>
                <th>window_size</th>
                <th>std_window</th>
                <th>Total</th>
                <th>Removed</th>
                <th>Remaining</th>
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

  const std_list = [];
  const stride = 20;
  for (let i = 0; i <= r_sorted.length - std_window; i += stride) {
    std_list.push(std(r_sorted.slice(i, i + std_window)));
  }

  const global_std = (std_list.length === 0 ? std(r_sorted) : median(std_list)) + 1e-12;
  const mask = Array(r_sorted.length).fill(true);

  if (r_sorted.length < window_size) {
    const m = mean(r_sorted);
    for (let i = 0; i < r_sorted.length; i++) {
      if (Math.abs(r_sorted[i] - m) > threshold * global_std) {
        mask[i] = false;
      }
    }
  } else {
    for (let i = 0; i <= r_sorted.length - window_size; i++) {
      const win = r_sorted.slice(i, i + window_size);
      const m = mean(win);
      for (let j = 0; j < window_size; j++) {
        if (Math.abs(win[j] - m) > threshold * global_std) {
          mask[i + j] = false;
        }
      }
    }
  }

  const x_f = [];
  const y_f = [];
  for (let k = 0; k < idx.length; k++) {
    if (mask[k]) {
      x_f.push(x[idx[k]]);
      y_f.push(y[idx[k]]);
    }
  }

  return { x: x_f, y: y_f };
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

function currentParams() {
  return {
    sigma: parseFloat(document.getElementById('sigma').value),
    n_points: parseInt(document.getElementById('n_points').value, 10),
    a: parseFloat(document.getElementById('a').value),
    b: parseFloat(document.getElementById('b').value),
    cluster_ratio: parseFloat(document.getElementById('cluster_ratio').value),
    near_ratio: parseFloat(document.getElementById('near_ratio').value),
    threshold: parseFloat(document.getElementById('threshold').value),
    window_size: parseInt(document.getElementById('window_size').value, 10),
    std_window: parseInt(document.getElementById('std_window').value, 10)
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
  document.getElementById('threshold_val').textContent = document.getElementById('threshold').value;
  document.getElementById('window_size_val').textContent = document.getElementById('window_size').value;
  document.getElementById('std_window_val').textContent = document.getElementById('std_window').value;
}

function renderScatter(scatter) {
  const traces = [
    {
      x: scatter.x_true_inlier,
      y: scatter.y_true_inlier,
      mode: 'markers',
      type: 'scatter',
      name: 'Points',
      marker: { size: 4, color: '#1f77b4' }
    },
    {
      x: scatter.x_true_outlier,
      y: scatter.y_true_outlier,
      mode: 'markers',
      type: 'scatter',
      name: 'True Outliers',
      marker: { size: 7, color: '#d62728' }
    },
    {
      x: scatter.x_detected_removed,
      y: scatter.y_detected_removed,
      mode: 'markers',
      type: 'scatter',
      name: 'Filtered Outliers',
      marker: { size: 7, color: '#2ca02c', symbol: 'x' }
    }
  ];

  Plotly.newPlot('plot_scatter', traces, {
    title: 'Point Distribution and Filtering Result',
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
      marker: { size: 6, color: 'orange' }
    }
  ];

  Plotly.newPlot('plot_rtheta', traces, {
    title: 'r-theta Plot',
    xaxis: { title: 'theta (radian)' },
    yaxis: { title: 'r' },
    legend: { orientation: 'h' }
  }, { responsive: true });
}

function renderResultsBox(data) {
  document.getElementById('result_box').innerHTML = `
    <strong>Method</strong><br><br>
    <strong>Outlier Removal:</strong> Proposed Local Z-Score<br><br>

    <strong>Total points:</strong> ${data.counts.total}<br>
    <strong>True outliers:</strong> ${data.counts.true_outlier_total}<br>
    <strong>Remaining after filtering:</strong> ${data.counts.remaining}<br>
    <strong>Removed:</strong> ${data.counts.removed}<br><br>

    <strong>threshold:</strong> ${data.params.threshold.toFixed(2)}<br>
    <strong>window_size:</strong> ${data.params.window_size}<br>
    <strong>std_window:</strong> ${data.params.std_window}
  `;
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
    <td>${params.threshold.toFixed(2)}</td>
    <td>${params.window_size}</td>
    <td>${params.std_window}</td>
    <td>${data.counts.total}</td>
    <td>${data.counts.removed}</td>
    <td>${data.counts.remaining}</td>
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

  const cluster_outliers = Math.floor(params.n_points * params.cluster_ratio);
  const near_ellipse_outliers = Math.floor(params.n_points * params.near_ratio);
  const random_outliers = 0;
  const random_seed = 42;

  const data = generateSyntheticEllipse({
    xc,
    yc,
    a: params.a,
    b: params.b,
    n_points: params.n_points,
    sigma: params.sigma,
    cluster_outliers,
    near_ellipse_outliers,
    random_outliers,
    random_seed
  });

  const cleaned = removeOutliersLocalZScoreProposed(
    data.X,
    data.Y,
    params.threshold,
    params.window_size,
    params.std_window
  );

  const splitDetected = splitDetectedOutliers(
    data.X,
    data.Y,
    cleaned.x,
    cleaned.y
  );

  const rtOrig = plotRThetaArrays(data.X, data.Y, xc, yc);
  const rtRemoved = splitDetected.removedX.length > 0
    ? plotRThetaArrays(splitDetected.removedX, splitDetected.removedY, xc, yc)
    : { theta: [], r: [] };

  return {
    params,
    counts: {
      total: data.X.length,
      remaining: cleaned.x.length,
      removed: data.X.length - cleaned.x.length,
      true_outlier_total: data.x_out_cluster.length + data.x_out_near.length + data.x_out_random.length
    },
    scatter: {
      x_true_inlier: data.x_in,
      y_true_inlier: data.y_in,
      x_true_outlier: [...data.x_out_cluster, ...data.x_out_near, ...data.x_out_random],
      y_true_outlier: [...data.y_out_cluster, ...data.y_out_near, ...data.y_out_random],
      x_detected_removed: splitDetected.removedX,
      y_detected_removed: splitDetected.removedY
    },
    rtheta: {
      theta_original: rtOrig.theta,
      r_original: rtOrig.r,
      theta_removed: rtRemoved.theta,
      r_removed: rtRemoved.r
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
    renderResultsBox(result);
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
