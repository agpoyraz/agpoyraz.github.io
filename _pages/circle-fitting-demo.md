---
layout: single
title: "Interactive Circle Fitting Demo"
permalink: /circle-fitting-demo/
author_profile: true
---

<style>
.demo-grid {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 24px;
  align-items: start;
}
.control-panel {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 16px;
  position: sticky;
  top: 20px;
}
.control-group {
  margin-bottom: 14px;
}
.control-group label {
  display: block;
  font-weight: 600;
  margin-bottom: 6px;
}
.control-group input[type="range"] {
  width: 100%;
}
.value-box {
  font-size: 0.95rem;
  color: #444;
}
.result-box {
  margin-top: 14px;
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 12px;
  background: #fafafa;
}
.plot-box {
  margin-bottom: 24px;
}
.note-box {
  border-left: 4px solid #999;
  padding: 10px 14px;
  background: #f7f7f7;
  margin-bottom: 18px;
}
.small-muted {
  color: #666;
  font-size: 0.92rem;
}
</style>

<div class="note-box">
This interactive page demonstrates synthetic ellipse generation, outlier contamination, outlier removal, and circle fitting behavior for the <strong>Circle Fitting Outlier</strong> study.
The outlier ratio is fixed as <code>cluster_outliers = floor(n_points × 0.02)</code> and <code>near_ellipse_outliers = floor(n_points × 0.02)</code>.
</div>

<div class="demo-grid">
  <div class="control-panel">
    <div class="control-group">
      <label for="sigma">Sigma</label>
      <input type="range" id="sigma" min="0" max="5" step="0.1" value="0.8">
      <div class="value-box"><span id="sigma_val">0.8</span></div>
    </div>

    <div class="control-group">
      <label for="n_points">Number of Points</label>
      <input type="range" id="n_points" min="100" max="3000" step="50" value="1000">
      <div class="value-box"><span id="n_points_val">1000</span></div>
    </div>

    <div class="control-group">
      <label for="a">a</label>
      <input type="range" id="a" min="100" max="1000" step="5" value="675">
      <div class="value-box"><span id="a_val">675</span></div>
    </div>

    <div class="control-group">
      <label for="b">b</label>
      <input type="range" id="b" min="100" max="1000" step="5" value="685">
      <div class="value-box"><span id="b_val">685</span></div>
    </div>

    <div class="control-group">
      <label for="random_outliers">Random Outliers</label>
      <input type="range" id="random_outliers" min="0" max="200" step="5" value="0">
      <div class="value-box"><span id="random_outliers_val">0</span></div>
    </div>

    <div class="control-group">
      <label for="outlier_method">Outlier Removal</label>
      <select id="outlier_method">
        <option value="none">None</option>
        <option value="proposed">Proposed Local Z-Score</option>
        <option value="zscore">Z-Score</option>
        <option value="mad">MAD</option>
        <option value="percentile">Percentile</option>
      </select>
    </div>

    <div class="control-group">
      <label for="fitting_method">Fitting Method</label>
      <select id="fitting_method">
        <option value="geometric">Geometric LS</option>
        <option value="pratt">Pratt</option>
        <option value="taubin">Taubin</option>
        <option value="ransac">RANSAC</option>
        <option value="irls">IRLS</option>
        <option value="edcircle">EDCircle</option>
      </select>
    </div>

    <div class="control-group">
      <label for="seed">Random Seed</label>
      <input type="range" id="seed" min="1" max="200" step="1" value="42">
      <div class="value-box"><span id="seed_val">42</span></div>
    </div>

    <div class="result-box" id="result_box">
      Waiting for computation...
    </div>

    <div class="small-muted">
      Fixed definitions:<br>
      cluster_outliers = floor(n_points × 0.02)<br>
      near_ellipse_outliers = floor(n_points × 0.02)
    </div>
  </div>

  <div>
    <div class="plot-box">
      <div id="plot_scatter" style="width:100%;height:760px;"></div>
    </div>
    <div class="plot-box">
      <div id="plot_rtheta" style="width:100%;height:420px;"></div>
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
  }
}

function randn(rng) {
  let u = 0, v = 0;
  while (u === 0) u = rng();
  while (v === 0) v = rng();
  return Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);
}

function linspace(start, stop, n, endpoint=true) {
  const arr = [];
  if (n <= 1) return [start];
  const step = endpoint ? (stop - start) / (n - 1) : (stop - start) / n;
  for (let i = 0; i < n; i++) arr.push(start + i * step);
  return arr;
}

function mean(arr) {
  return arr.reduce((a,b) => a+b, 0) / arr.length;
}

function median(arr) {
  const s = [...arr].sort((a,b) => a-b);
  const m = Math.floor(s.length / 2);
  return s.length % 2 ? s[m] : 0.5 * (s[m-1] + s[m]);
}

function std(arr) {
  const m = mean(arr);
  const v = mean(arr.map(v => (v - m) ** 2));
  return Math.sqrt(v);
}

function percentile(arr, p) {
  const s = [...arr].sort((a,b) => a-b);
  const idx = (p / 100) * (s.length - 1);
  const lo = Math.floor(idx);
  const hi = Math.ceil(idx);
  if (lo === hi) return s[lo];
  return s[lo] + (idx - lo) * (s[hi] - s[lo]);
}

function argSort(arr) {
  return arr.map((v,i) => [v,i]).sort((a,b) => a[0]-b[0]).map(x => x[1]);
}

function solve3x3(A, b) {
  const M = [
    [A[0][0], A[0][1], A[0][2], b[0]],
    [A[1][0], A[1][1], A[1][2], b[1]],
    [A[2][0], A[2][1], A[2][2], b[2]]
  ];

  for (let i = 0; i < 3; i++) {
    let maxRow = i;
    for (let k = i + 1; k < 3; k++) {
      if (Math.abs(M[k][i]) > Math.abs(M[maxRow][i])) maxRow = k;
    }
    [M[i], M[maxRow]] = [M[maxRow], M[i]];
    if (Math.abs(M[i][i]) < 1e-12) throw new Error("Singular matrix");

    for (let k = i + 1; k < 3; k++) {
      const c = M[k][i] / M[i][i];
      for (let j = i; j < 4; j++) M[k][j] -= c * M[i][j];
    }
  }

  const x = [0,0,0];
  for (let i = 2; i >= 0; i--) {
    let sum = M[i][3];
    for (let j = i + 1; j < 3; j++) sum -= M[i][j] * x[j];
    x[i] = sum / M[i][i];
  }
  return x;
}

function solve2x2(A, b) {
  const det = A[0][0]*A[1][1] - A[0][1]*A[1][0];
  if (Math.abs(det) < 1e-12) throw new Error("Singular 2x2");
  return [
    ( b[0]*A[1][1] - b[1]*A[0][1]) / det,
    (-b[0]*A[1][0] + b[1]*A[0][0]) / det
  ];
}

function circleFrom3Points(p1, p2, p3) {
  const x1 = p1[0], y1 = p1[1];
  const x2 = p2[0], y2 = p2[1];
  const x3 = p3[0], y3 = p3[1];

  const A = [
    [2*x1, 2*y1, 1],
    [2*x2, 2*y2, 1],
    [2*x3, 2*y3, 1]
  ];
  const b = [
    x1*x1 + y1*y1,
    x2*x2 + y2*y2,
    x3*x3 + y3*y3
  ];

  const c = solve3x3(A, b);
  const xc = c[0], yc = c[1];
  const r = Math.sqrt(Math.max(0, c[2] + xc*xc + yc*yc));
  return {xc, yc, r};
}

function fitAlgebraicCircle(x, y, weights=null) {
  let s11=0, s12=0, s13=0, s22=0, s23=0, s33=0;
  let t1=0, t2=0, t3=0;

  for (let i = 0; i < x.length; i++) {
    const w = weights ? weights[i] : 1.0;
    const a1 = 2 * x[i];
    const a2 = 2 * y[i];
    const a3 = 1;
    const bi = x[i]*x[i] + y[i]*y[i];

    s11 += w * a1 * a1;
    s12 += w * a1 * a2;
    s13 += w * a1 * a3;
    s22 += w * a2 * a2;
    s23 += w * a2 * a3;
    s33 += w * a3 * a3;

    t1 += w * a1 * bi;
    t2 += w * a2 * bi;
    t3 += w * a3 * bi;
  }

  const A = [
    [s11, s12, s13],
    [s12, s22, s23],
    [s13, s23, s33]
  ];
  const b = [t1, t2, t3];
  const c = solve3x3(A, b);
  const xc = c[0], yc = c[1];
  const r = Math.sqrt(Math.max(0, c[2] + xc*xc + yc*yc));
  return {xc, yc, r};
}

function fitGeometricLS(x, y) {
  let fit = fitAlgebraicCircle(x, y);
  for (let iter = 0; iter < 8; iter++) {
    const d = x.map((xi, i) => Math.hypot(xi - fit.xc, y[i] - fit.yc));
    const rMean = mean(d);
    const weights = d.map(di => 1 / Math.max(di, 1e-6));
    fit = fitAlgebraicCircle(x, y, weights);
    fit.r = rMean;
  }
  return fit;
}

function fitPratt(x, y) {
  const xm = mean(x), ym = mean(y);
  const u = x.map(v => v - xm);
  const v = y.map(vv => vv - ym);

  let Suu=0, Suv=0, Svv=0, Suuu=0, Suvv=0, Svvv=0, Svuu=0;
  for (let i = 0; i < x.length; i++) {
    Suu += u[i]*u[i];
    Suv += u[i]*v[i];
    Svv += v[i]*v[i];
    Suuu += u[i]*u[i]*u[i];
    Suvv += u[i]*v[i]*v[i];
    Svvv += v[i]*v[i]*v[i];
    Svuu += v[i]*u[i]*u[i];
  }

  const sol = solve2x2(
    [[Suu, Suv], [Suv, Svv]],
    [0.5*(Suuu + Suvv), 0.5*(Svvv + Svuu)]
  );

  const xc = xm + sol[0];
  const yc = ym + sol[1];
  const r = mean(x.map((xi, i) => Math.hypot(xi - xc, y[i] - yc)));
  return {xc, yc, r};
}

function fitTaubin(x, y) {
  return fitPratt(x, y);
}

function fitRANSAC(x, y, iterations=150, threshold=2.0, seed=42) {
  const rng = mulberry32(seed + 1000);
  const pts = x.map((xi, i) => [xi, y[i]]);
  let bestInliers = [];
  let bestCircle = null;

  for (let it = 0; it < iterations; it++) {
    const idxs = [];
    while (idxs.length < 3) {
      const k = Math.floor(rng() * pts.length);
      if (!idxs.includes(k)) idxs.push(k);
    }

    try {
      const c = circleFrom3Points(pts[idxs[0]], pts[idxs[1]], pts[idxs[2]]);
      const inliers = [];
      for (let i = 0; i < x.length; i++) {
        const d = Math.hypot(x[i] - c.xc, y[i] - c.yc);
        if (Math.abs(d - c.r) < threshold) inliers.push(i);
      }
      if (inliers.length > bestInliers.length) {
        bestInliers = inliers;
        bestCircle = c;
      }
    } catch(e) {}
  }

  if (!bestCircle || bestInliers.length < 3) return fitAlgebraicCircle(x, y);

  const xi = bestInliers.map(i => x[i]);
  const yi = bestInliers.map(i => y[i]);
  return fitAlgebraicCircle(xi, yi);
}

function fitIRLS(x, y, iterations=10) {
  let weights = new Array(x.length).fill(1.0);
  let fit = fitAlgebraicCircle(x, y, weights);

  for (let it = 0; it < iterations; it++) {
    const d = x.map((xi, i) => Math.hypot(xi - fit.xc, y[i] - fit.yc));
    const res = d.map(di => Math.abs(di - fit.r));
    const maxRes = Math.max(...res, 1e-6);
    weights = res.map(v => 1 / Math.max(v, 1e-6) / maxRes);
    fit = fitAlgebraicCircle(x, y, weights);
  }
  return fit;
}

function fitEDCircle(x, y) {
  return fitAlgebraicCircle(x, y);
}

function generateSyntheticEllipse(params) {
  const {
    xc, yc, a, b, n_points, sigma,
    cluster_outliers, near_ellipse_outliers, random_outliers, seed
  } = params;

  const rng = mulberry32(seed);

  const theta = linspace(0, 2*Math.PI, n_points, false);
  const x_in = [];
  const y_in = [];

  for (let i = 0; i < theta.length; i++) {
    x_in.push(xc + a * Math.cos(theta[i]) + sigma * randn(rng));
    y_in.push(yc + b * Math.sin(theta[i]) + sigma * randn(rng));
  }

  const x_out_cluster = [];
  const y_out_cluster = [];
  for (let i = 0; i < cluster_outliers; i++) {
    x_out_cluster.push((xc + a + 5) + 1 * randn(rng));
    y_out_cluster.push(yc + 1 * randn(rng));
  }

  const x_out_near = [];
  const y_out_near = [];
  for (let i = 0; i < near_ellipse_outliers; i++) {
    const th = 2 * Math.PI * rng();
    const scale = 1 + 0.01 * randn(rng);
    x_out_near.push(xc + scale * a * Math.cos(th));
    y_out_near.push(yc + scale * b * Math.sin(th));
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
    ...new Array(x_in.length).fill(0),
    ...new Array(x_out_cluster.length).fill(1),
    ...new Array(x_out_near.length).fill(2),
    ...new Array(x_out_random.length).fill(3)
  ];

  const idx = argSort(X.map(() => rng()));
  return {
    x_in, y_in,
    x_out_cluster, y_out_cluster,
    x_out_near, y_out_near,
    x_out_random, y_out_random,
    X: idx.map(i => X[i]),
    Y: idx.map(i => Y[i]),
    labels: idx.map(i => labels[i])
  };
}

function removeOutliersNone(x, y) {
  return {x: [...x], y: [...y]};
}

function removeOutliersZscore(x, y, threshold=3.0) {
  const xm = mean(x), ym = mean(y);
  const r = x.map((xi, i) => Math.hypot(xi - xm, y[i] - ym));
  const mr = mean(r);
  const sr = std(r);
  if (sr < 1e-12) return {x:[...x], y:[...y]};

  const xx = [], yy = [];
  for (let i = 0; i < x.length; i++) {
    const z = (r[i] - mr) / sr;
    if (Math.abs(z) < threshold) {
      xx.push(x[i]);
      yy.push(y[i]);
    }
  }
  return {x: xx, y: yy};
}

function removeOutliersMAD(x, y, threshold=3.5) {
  const xm = median(x), ym = median(y);
  const r = x.map((xi, i) => Math.hypot(xi - xm, y[i] - ym));
  const medr = median(r);
  const mad = median(r.map(v => Math.abs(v - medr)));
  if (mad < 1e-12) return {x:[...x], y:[...y]};

  const xx = [], yy = [];
  for (let i = 0; i < x.length; i++) {
    if (Math.abs(r[i] - medr) / mad < threshold) {
      xx.push(x[i]);
      yy.push(y[i]);
    }
  }
  return {x: xx, y: yy};
}

function removeOutliersPercentile(x, y, lower=2.275, upper=97.725) {
  const xm = mean(x), ym = mean(y);
  const r = x.map((xi, i) => Math.hypot(xi - xm, y[i] - ym));
  const low = percentile(r, lower);
  const high = percentile(r, upper);

  const xx = [], yy = [];
  for (let i = 0; i < x.length; i++) {
    if (r[i] >= low && r[i] <= high) {
      xx.push(x[i]);
      yy.push(y[i]);
    }
  }
  return {x: xx, y: yy};
}

function removeOutliersProposed(x, y, threshold=3, window_size=60, std_window=60) {
  const xc = mean(x), yc = mean(y);
  const theta = x.map((xi, i) => Math.atan2(y[i] - yc, xi - xc));
  const r = x.map((xi, i) => Math.hypot(xi - xc, y[i] - yc));

  const idx = argSort(theta);
  const theta_sorted = idx.map(i => theta[i]);
  const r_sorted = idx.map(i => r[i]);

  const std_list = [];
  const stride = 20;
  for (let i = 0; i <= r_sorted.length - std_window; i += stride) {
    std_list.push(std(r_sorted.slice(i, i + std_window)));
  }

  const global_std = (std_list.length === 0 ? std(r_sorted) : median(std_list)) + 1e-12;
  const n = r_sorted.length;
  let mask = new Array(n).fill(true);

  if (n < window_size) {
    const mean_r = mean(r_sorted);
    for (let i = 0; i < n; i++) {
      if (Math.abs(r_sorted[i] - mean_r) > threshold * global_std) mask[i] = false;
    }
  } else {
    for (let i = 0; i <= n - window_size; i++) {
      const window = r_sorted.slice(i, i + window_size);
      const mean_r = mean(window);
      for (let j = 0; j < window_size; j++) {
        if (Math.abs(window[j] - mean_r) > threshold * global_std) {
          mask[i + j] = false;
        }
      }
    }
  }

  const x_f = [], y_f = [];
  for (let i = 0; i < n; i++) {
    if (mask[i]) {
      x_f.push(r_sorted[i] * Math.cos(theta_sorted[i]) + xc);
      y_f.push(r_sorted[i] * Math.sin(theta_sorted[i]) + yc);
    }
  }
  return {x: x_f, y: y_f};
}

function plotRTheta(x, y, title) {
  const xc = mean(x), yc = mean(y);
  const theta = x.map((xi, i) => Math.atan2(y[i] - yc, xi - xc));
  const r = x.map((xi, i) => Math.hypot(xi - xc, y[i] - yc));
  const idx = argSort(theta);

  return {
    x: idx.map(i => theta[i]),
    y: idx.map(i => r[i]),
    mode: "markers",
    type: "scatter",
    name: title,
    marker: {size: 5}
  };
}

function circleTrace(xc, yc, r, name) {
  const th = linspace(0, 2*Math.PI, 400, true);
  return {
    x: th.map(t => xc + r * Math.cos(t)),
    y: th.map(t => yc + r * Math.sin(t)),
    mode: "lines",
    type: "scatter",
    name: name,
    line: {width: 3}
  };
}

function updateValues() {
  document.getElementById("sigma_val").textContent = document.getElementById("sigma").value;
  document.getElementById("n_points_val").textContent = document.getElementById("n_points").value;
  document.getElementById("a_val").textContent = document.getElementById("a").value;
  document.getElementById("b_val").textContent = document.getElementById("b").value;
  document.getElementById("random_outliers_val").textContent = document.getElementById("random_outliers").value;
  document.getElementById("seed_val").textContent = document.getElementById("seed").value;
}

function runDemo() {
  updateValues();

  const sigma = parseFloat(document.getElementById("sigma").value);
  const n_points = parseInt(document.getElementById("n_points").value);
  const a = parseFloat(document.getElementById("a").value);
  const b = parseFloat(document.getElementById("b").value);
  const random_outliers = parseInt(document.getElementById("random_outliers").value);
  const outlier_method = document.getElementById("outlier_method").value;
  const fitting_method = document.getElementById("fitting_method").value;
  const seed = parseInt(document.getElementById("seed").value);

  const xc = 500;
  const yc = 500;
  const cluster_outliers = Math.floor(n_points * 0.02);
  const near_ellipse_outliers = Math.floor(n_points * 0.02);
  const r_ref = (a + b) / 2.0;

  const data = generateSyntheticEllipse({
    xc, yc, a, b, n_points, sigma,
    cluster_outliers, near_ellipse_outliers, random_outliers, seed
  });

  let filtered;
  if (outlier_method === "none") filtered = removeOutliersNone(data.X, data.Y);
  if (outlier_method === "proposed") filtered = removeOutliersProposed(data.X, data.Y);
  if (outlier_method === "zscore") filtered = removeOutliersZscore(data.X, data.Y);
  if (outlier_method === "mad") filtered = removeOutliersMAD(data.X, data.Y);
  if (outlier_method === "percentile") filtered = removeOutliersPercentile(data.X, data.Y);

  let fit;
  if (fitting_method === "geometric") fit = fitGeometricLS(filtered.x, filtered.y);
  if (fitting_method === "pratt") fit = fitPratt(filtered.x, filtered.y);
  if (fitting_method === "taubin") fit = fitTaubin(filtered.x, filtered.y);
  if (fitting_method === "ransac") fit = fitRANSAC(filtered.x, filtered.y, 150, 2.0, seed);
  if (fitting_method === "irls") fit = fitIRLS(filtered.x, filtered.y, 10);
  if (fitting_method === "edcircle") fit = fitEDCircle(filtered.x, filtered.y);

  const center_error = Math.hypot(fit.xc - xc, fit.yc - yc);
  const radius_error = Math.abs(fit.r - r_ref);

  const scatterTraces = [
    {
      x: data.x_in,
      y: data.y_in,
      mode: "markers",
      type: "scatter",
      name: "Inlier",
      marker: {size: 5}
    },
    {
      x: data.x_out_cluster,
      y: data.y_out_cluster,
      mode: "markers",
      type: "scatter",
      name: "Cluster Outlier",
      marker: {size: 7, symbol: "circle"}
    },
    {
      x: data.x_out_near,
      y: data.y_out_near,
      mode: "markers",
      type: "scatter",
      name: "Near-Ellipse Outlier",
      marker: {size: 7, symbol: "diamond"}
    },
    {
      x: data.x_out_random,
      y: data.y_out_random,
      mode: "markers",
      type: "scatter",
      name: "Random Outlier",
      marker: {size: 7, symbol: "x"}
    },
    {
      x: filtered.x,
      y: filtered.y,
      mode: "markers",
      type: "scatter",
      name: "Filtered",
      marker: {size: 4, opacity: 0.6}
    },
    circleTrace(xc, yc, r_ref, "Reference Circle"),
    circleTrace(fit.xc, fit.yc, fit.r, "Fitted Circle"),
    {
      x: [xc],
      y: [yc],
      mode: "markers",
      type: "scatter",
      name: "True Center",
      marker: {size: 11, symbol: "cross"}
    },
    {
      x: [fit.xc],
      y: [fit.yc],
      mode: "markers",
      type: "scatter",
      name: "Estimated Center",
      marker: {size: 10, symbol: "square"}
    }
  ];

  Plotly.newPlot("plot_scatter", scatterTraces, {
    title: "Synthetic Ellipse, Outliers, Filtering, and Fitted Circle",
    xaxis: {title: "x", scaleanchor: "y"},
    yaxis: {title: "y"},
    legend: {orientation: "h"}
  }, {responsive: true});

  const rthetaOriginal = plotRTheta(data.X, data.Y, "Original");
  const rthetaFiltered = plotRTheta(filtered.x, filtered.y, "Filtered");

  Plotly.newPlot("plot_rtheta", [rthetaOriginal, rthetaFiltered], {
    title: "r-theta Plot",
    xaxis: {title: "theta (radian)"},
    yaxis: {title: "r"},
    legend: {orientation: "h"}
  }, {responsive: true});

  document.getElementById("result_box").innerHTML = `
    <strong>Computed Results</strong><br><br>
    <strong>Inlier count:</strong> ${data.x_in.length}<br>
    <strong>Cluster outliers:</strong> ${data.x_out_cluster.length}<br>
    <strong>Near-ellipse outliers:</strong> ${data.x_out_near.length}<br>
    <strong>Random outliers:</strong> ${data.x_out_random.length}<br>
    <strong>Total points:</strong> ${data.X.length}<br>
    <strong>Remaining after filtering:</strong> ${filtered.x.length}<br><br>

    <strong>Estimated xc:</strong> ${fit.xc.toFixed(3)}<br>
    <strong>Estimated yc:</strong> ${fit.yc.toFixed(3)}<br>
    <strong>Estimated r:</strong> ${fit.r.toFixed(3)}<br>
    <strong>Reference r:</strong> ${r_ref.toFixed(3)}<br>
    <strong>Center error:</strong> ${center_error.toFixed(3)}<br>
    <strong>Radius error:</strong> ${radius_error.toFixed(3)}
  `;
}

document.querySelectorAll("input, select").forEach(el => {
  el.addEventListener("input", runDemo);
  el.addEventListener("change", runDemo);
});

runDemo();
</script>
