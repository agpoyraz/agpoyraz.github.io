---
layout: single
title: "Interactive Circle Fitting Demo"
permalink: /circle-fitting-demo/
author_profile: true
---

<style>
  .cf-grid {
    display: grid;
    grid-template-columns: 320px 1fr;
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
  .cf-group { margin-bottom: 16px; }
  .cf-group label {
    display: block;
    font-weight: 600;
    margin-bottom: 6px;
  }
  .cf-group input[type="range"],
  .cf-group select,
  .cf-group button {
    width: 100%;
  }
  .cf-value { font-size: 0.95rem; color: #555; }
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
  .cf-plot { margin-bottom: 24px; }
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
  .cf-table th, .cf-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  .cf-table th { background: #f2f2f2; }
  .cf-small { font-size: 0.9rem; color: #666; }
  @media (max-width: 980px) {
    .cf-grid { grid-template-columns: 1fr; }
    .cf-panel { position: static; }
  }
</style>

<div class="cf-note">
  This page runs the circle fitting outlier demo directly in the browser with <strong>Pyodide</strong>. The synthetic data generation and analysis logic follows the uploaded Python implementation. The user-adjustable variables are <code>sigma</code>, <code>n_points</code>, <code>a</code>, and <code>b</code>. The cluster outlier count is fixed as <code>int(n_points * 0.02)</code>.
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
      <label for="selected_outlier">Visualized Outlier Method</label>
      <select id="selected_outlier">
        <option>None</option>
        <option>Proposed Local Z-Score</option>
        <option>Z-Score</option>
        <option>MAD</option>
        <option>DBSCAN</option>
        <option>LOF</option>
        <option>Percentile</option>
      </select>
    </div>

    <div class="cf-group">
      <label for="selected_fitting">Visualized Fitting Method</label>
      <select id="selected_fitting">
        <option>Geometric LS</option>
        <option>Pratt</option>
        <option>Taubin</option>
        <option>RANSAC</option>
        <option>IRLS</option>
        <option>Hyper LS</option>
        <option>M-Estimator</option>
        <option>LMedS</option>
        <option>TLS</option>
        <option>Bayesian</option>
        <option>Gradient Descent</option>
        <option>EDCircle</option>
      </select>
    </div>

    <div class="cf-group">
      <button id="run_btn">Run Demo</button>
    </div>

    <div class="cf-small">
      Fixed settings:<br>
      cluster_outliers = int(n_points × 0.02)<br>
      near_ellipse_outliers = int(n_points × 0.02)<br>
      random_outliers = 0<br>
      random_seed = 42
    </div>

    <div class="cf-status" id="status_box">Loading Python runtime...</div>

    <div class="cf-box" id="result_box">
      Results will appear here.
    </div>
  </div>

  <div>
    <div class="cf-plot"><div id="plot_scatter" style="width:100%;height:760px;"></div></div>
    <div class="cf-plot"><div id="plot_rtheta" style="width:100%;height:420px;"></div></div>
    <div class="cf-box">
      <h3 style="margin-top:0;">Top 10 Results</h3>
      <div class="cf-table-wrap">
        <table class="cf-table" id="results_table">
          <thead>
            <tr>
              <th>Outlier Method</th>
              <th>Fitting Method</th>
              <th>Remain</th>
              <th>Removed</th>
              <th>xc</th>
              <th>yc</th>
              <th>r</th>
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

<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>
<script src="https://cdn.jsdelivr.net/pyodide/v0.27.2/full/pyodide.js"></script>

<script>
let pyodide = null;
let pyReady = false;

const pythonSource = `
import json
import random
import numpy as np
from scipy.optimize import least_squares
from sklearn.cluster import DBSCAN
from sklearn.neighbors import LocalOutlierFactor
from sklearn.preprocessing import StandardScaler


def generate_synthetic_ellipse(
    xc=100,
    yc=80,
    a=50,
    b=50,
    n_points=200,
    sigma=0.8,
    cluster_outliers=10,
    near_ellipse_outliers=10,
    random_outliers=10,
    random_seed=None
):
    if random_seed is not None:
        np.random.seed(random_seed)
        random.seed(random_seed)

    theta = np.linspace(0, 2 * np.pi, n_points, endpoint=False)
    x = xc + a * np.cos(theta)
    y = yc + b * np.sin(theta)

    x_in = x + sigma * np.random.randn(len(x))
    y_in = y + sigma * np.random.randn(len(y))

    if cluster_outliers > 0:
        x_out_cluster = (xc + a + 5) + 1 * np.random.randn(cluster_outliers)
        y_out_cluster = yc + 1 * np.random.randn(cluster_outliers)
    else:
        x_out_cluster = np.array([])
        y_out_cluster = np.array([])

    if near_ellipse_outliers > 0:
        theta_o = 2 * np.pi * np.random.rand(near_ellipse_outliers)
        scale = 1 + 0.01 * np.random.randn(near_ellipse_outliers)
        x_out_near = xc + scale * a * np.cos(theta_o)
        y_out_near = yc + scale * b * np.sin(theta_o)
    else:
        x_out_near = np.array([])
        y_out_near = np.array([])

    if random_outliers > 0:
        span = 1.5 * max(a, b) * 2
        x_out_random = (xc - span) + (2 * span) * np.random.rand(random_outliers)
        y_out_random = (yc - span) + (2 * span) * np.random.rand(random_outliers)
    else:
        x_out_random = np.array([])
        y_out_random = np.array([])

    x_out = np.concatenate([x_out_cluster, x_out_near, x_out_random])
    y_out = np.concatenate([y_out_cluster, y_out_near, y_out_random])

    X = np.concatenate([x_in, x_out])
    Y = np.concatenate([y_in, y_out])

    labels = np.concatenate([
        np.zeros(len(x_in), dtype=int),
        np.ones(len(x_out_cluster), dtype=int),
        2 * np.ones(len(x_out_near), dtype=int),
        3 * np.ones(len(x_out_random), dtype=int)
    ])

    idx = np.random.permutation(len(X))
    X = X[idx]
    Y = Y[idx]
    labels = labels[idx]

    return (
        x_in, y_in,
        x_out_cluster, y_out_cluster,
        x_out_near, y_out_near,
        x_out_random, y_out_random,
        X, Y, labels
    )


def remove_outliers_local_zscore_proposed(x, y, threshold=3, window_size=60, std_window=60):
    x = np.asarray(x).flatten()
    y = np.asarray(y).flatten()

    xc = np.mean(x)
    yc = np.mean(y)

    theta = np.arctan2(y - yc, x - xc)
    r = np.sqrt((x - xc)**2 + (y - yc)**2)

    idx = np.argsort(theta)
    theta_sorted = theta[idx]
    r_sorted = r[idx]

    std_list = []
    stride = 20
    for i in range(0, len(r_sorted) - std_window + 1, stride):
        std_list.append(np.std(r_sorted[i:i + std_window]))

    if len(std_list) == 0:
        global_std = np.std(r_sorted) + 1e-12
    else:
        global_std = np.median(std_list) + 1e-12

    n = len(r_sorted)
    mask = np.ones(n, dtype=bool)

    if n < window_size:
        mean_r = np.mean(r_sorted)
        outliers = np.abs(r_sorted - mean_r) > threshold * global_std
        mask = ~outliers
    else:
        for i in range(n - window_size + 1):
            window = r_sorted[i:i + window_size]
            mean_r = np.mean(window)
            outliers = np.abs(window - mean_r) > threshold * global_std
            mask[i:i + window_size] &= ~outliers

    r_clean = r_sorted[mask]
    theta_clean = theta_sorted[mask]

    x_filt = r_clean * np.cos(theta_clean) + xc
    y_filt = r_clean * np.sin(theta_clean) + yc

    return x_filt, y_filt


def remove_outliers_zscore(x, y, threshold=3.0):
    x = np.asarray(x)
    y = np.asarray(y)

    r = np.sqrt((x - np.mean(x))**2 + (y - np.mean(y))**2)
    std_r = np.std(r)

    if std_r < 1e-12:
        return x, y

    z = (r - np.mean(r)) / std_r
    mask = np.abs(z) < threshold
    return x[mask], y[mask]


def remove_outliers_mad(x, y, threshold=3.5):
    x = np.asarray(x)
    y = np.asarray(y)

    r = np.sqrt((x - np.median(x))**2 + (y - np.median(y))**2)
    med_r = np.median(r)
    mad = np.median(np.abs(r - med_r))

    if mad < 1e-12:
        return x, y

    mask = np.abs(r - med_r) / mad < threshold
    return x[mask], y[mask]


def remove_outliers_dbscan(x, y, eps=0.3, min_samples=5):
    x = np.asarray(x)
    y = np.asarray(y)

    coords = np.column_stack((x, y))
    coords_scaled = StandardScaler().fit_transform(coords)

    db = DBSCAN(eps=eps, min_samples=min_samples).fit(coords_scaled)
    mask = db.labels_ != -1
    return x[mask], y[mask]


def remove_outliers_lof(x, y, n_neighbors=20):
    x = np.asarray(x)
    y = np.asarray(y)

    coords = np.column_stack((x, y))

    if len(x) < n_neighbors:
        return x, y

    lof = LocalOutlierFactor(n_neighbors=n_neighbors)
    mask = lof.fit_predict(coords) == 1
    return x[mask], y[mask]


def remove_outliers_percentile(x, y, lower=2.275, upper=97.725):
    x = np.asarray(x)
    y = np.asarray(y)

    r = np.sqrt((x - np.mean(x))**2 + (y - np.mean(y))**2)
    low, high = np.percentile(r, [lower, upper])
    mask = (r >= low) & (r <= high)
    return x[mask], y[mask]


# --- Fitting Yöntemleri ---

def fit_geometric_ls(x, y):
    def residuals(c, x, y):
        Ri = np.sqrt((x - c[0])**2 + (y - c[1])**2)
        return Ri - Ri.mean()
    x_m = np.mean(x)
    y_m = np.mean(y)
    result = least_squares(residuals, x0=[x_m, y_m], args=(x, y))
    x0, y0 = result.x
    r = np.mean(np.sqrt((x - x0)**2 + (y - y0)**2))
    return x0, y0, r


def fit_pratt(x, y):
    x = np.array(x)
    y = np.array(y)
    x_m = np.mean(x)
    y_m = np.mean(y)
    u = x - x_m
    v = y - y_m
    Suu = np.sum(u**2)
    Suv = np.sum(u*v)
    Svv = np.sum(v**2)
    Suuu = np.sum(u**3)
    Suvv = np.sum(u*v**2)
    Svvv = np.sum(v**3)
    Svuu = np.sum(v*u**2)
    A = np.array([[Suu, Suv], [Suv, Svv]])
    b = np.array([0.5 * (Suuu + Suvv), 0.5 * (Svvv + Svuu)])
    uc, vc = np.linalg.solve(A, b)
    x0 = x_m + uc
    y0 = y_m + vc
    r = np.mean(np.sqrt((x - x0)**2 + (y - y0)**2))
    return x0, y0, r


def fit_taubin(x, y):
    x = np.array(x)
    y = np.array(y)
    x_m = np.mean(x)
    y_m = np.mean(y)
    u = x - x_m
    v = y - y_m
    Suu = np.sum(u**2)
    Suv = np.sum(u*v)
    Svv = np.sum(v**2)
    Suuu = np.sum(u**3)
    Suvv = np.sum(u*v**2)
    Svvv = np.sum(v**3)
    Svuu = np.sum(v*u**2)
    A = np.array([[Suu, Suv], [Suv, Svv]])
    B = np.array([Suuu + Suvv, Svvv + Svuu]) / 2
    uc, vc = np.linalg.solve(A, B)
    x0 = x_m + uc
    y0 = y_m + vc
    r = np.sqrt(uc**2 + vc**2 + (Suu + Svv) / len(x))
    return x0, y0, r


def fit_ransac(x, y, iterations=100, threshold=2.0):
    best_inliers = []
    best_circle = (0, 0, 0)
    x = np.array(x)
    y = np.array(y)
    points = np.stack([x, y], axis=1)

    for _ in range(iterations):
        samples = points[random.sample(range(len(points)), 3)]
        try:
            A = np.c_[2*samples[:,0], 2*samples[:,1], np.ones(3)]
            b = samples[:,0]**2 + samples[:,1]**2
            c = np.linalg.lstsq(A, b, rcond=None)[0]
            xc, yc = c[0], c[1]
            r = np.sqrt(c[2] + xc**2 + yc**2)
            d = np.sqrt((x - xc)**2 + (y - yc)**2)
            inliers = d[np.abs(d - r) < threshold]
            if len(inliers) > len(best_inliers):
                best_inliers = inliers
                best_circle = (xc, yc, r)
        except:
            continue
    return best_circle


def fit_irls(x, y, iterations=10):
    x = np.array(x)
    y = np.array(y)
    weights = np.ones_like(x)
    for _ in range(iterations):
        A = np.c_[2*x, 2*y, np.ones(x.shape[0])]
        b = x**2 + y**2
        W = np.diag(weights)
        Aw = W @ A
        bw = W @ b
        c = np.linalg.lstsq(Aw, bw, rcond=None)[0]
        x0, y0 = c[0], c[1]
        r = np.sqrt(c[2] + x0**2 + y0**2)
        d = np.sqrt((x - x0)**2 + (y - y0)**2)
        weights = 1.0 / np.maximum(np.abs(d - r), 1e-6)
        weights /= np.max(weights)
    return x0, y0, r


def fit_hyper_ls(x, y):
    x = np.array(x)
    y = np.array(y)
    D = np.column_stack((x * x + y * y, x, y, np.ones_like(x)))
    S = np.dot(D.T, D)
    C = np.zeros((4, 4))
    C[0, 3] = C[3, 0] = 2
    C[1, 1] = C[2, 2] = 1

    try:
        eigvals, eigvecs = np.linalg.eig(np.linalg.inv(S) @ C)
        cond = np.isreal(eigvals)
        eigvec = eigvecs[:, cond][:, 0].real
        A, B, C_, D_ = eigvec
        x0 = -B / (2 * A)
        y0 = -C_ / (2 * A)
        r = np.sqrt((B**2 + C_**2 - 4 * A * D_) / (4 * A**2))
    except:
        x0, y0, r = 0, 0, 0

    return x0, y0, r


def fit_m_estimator(x, y, iterations=10, delta=1.0):
    x = np.array(x)
    y = np.array(y)
    weights = np.ones_like(x)
    for _ in range(iterations):
        A = np.c_[2*x, 2*y, np.ones_like(x)]
        b = x**2 + y**2
        W = np.diag(weights)
        try:
            c = np.linalg.lstsq(W @ A, W @ b, rcond=None)[0]
        except:
            break
        x0, y0 = c[0], c[1]
        r = np.sqrt(c[2] + x0**2 + y0**2)
        d = np.sqrt((x - x0)**2 + (y - y0)**2)
        res = np.abs(d - r)
        weights = np.where(res <= delta, 1, delta / res)
    return x0, y0, r


def fit_lmeds(x, y):
    x = np.array(x)
    y = np.array(y)
    points = np.stack([x, y], axis=1)
    best_median = np.inf
    best_circle = (0, 0, 0)

    for _ in range(100):
        sample = points[random.sample(range(len(points)), 3)]
        try:
            A = np.c_[2*sample[:, 0], 2*sample[:, 1], np.ones(3)]
            b = sample[:, 0]**2 + sample[:, 1]**2
            c = np.linalg.lstsq(A, b, rcond=None)[0]
            xc, yc = c[0], c[1]
            r = np.sqrt(c[2] + xc**2 + yc**2)
            d = np.sqrt((x - xc)**2 + (y - yc)**2)
            residuals = np.abs(d - r)
            median_residual = np.median(residuals)
            if median_residual < best_median:
                best_median = median_residual
                best_circle = (xc, yc, r)
        except:
            continue

    return best_circle


def fit_tls(x, y):
    x = np.asarray(x, dtype=float)
    y = np.asarray(y, dtype=float)

    A = np.c_[2 * x, 2 * y, np.ones_like(x)]
    b = x**2 + y**2
    M = np.column_stack((A, b))

    _, _, Vt = np.linalg.svd(M)
    v = Vt[-1, :]

    if abs(v[3]) < 1e-12:
        raise ValueError("TLS çözümü kararsız: v[3] çok küçük.")

    p = -v[:3] / v[3]

    x0 = p[0]
    y0 = p[1]
    c0 = p[2]

    radicand = c0 + x0**2 + y0**2
    if radicand < 0:
        raise ValueError("Negatif yarıçap karesi oluştu.")

    r = np.sqrt(radicand)
    return x0, y0, r


def fit_bayesian(x, y):
    x = np.array(x)
    y = np.array(y)
    x0 = np.mean(x)
    y0 = np.mean(y)
    r = np.mean(np.sqrt((x - x0)**2 + (y - y0)**2))
    noise = np.random.normal(0, 0.5, 100)
    r_samples = r + noise
    r_mean = np.mean(r_samples)
    return x0, y0, r_mean


def fit_gradient_descent(x, y, lr=1e-3, iterations=1000):
    x = np.array(x)
    y = np.array(y)
    x0, y0 = np.mean(x), np.mean(y)
    r = np.mean(np.sqrt((x - x0)**2 + (y - y0)**2))

    for _ in range(iterations):
        d = np.sqrt((x - x0)**2 + (y - y0)**2)
        dr = d - r
        dx0 = np.mean((x0 - x) * dr / d)
        dy0 = np.mean((y0 - y) * dr / d)
        dr0 = -np.mean(dr)
        x0 -= lr * dx0
        y0 -= lr * dy0
        r -= lr * dr0

    return x0, y0, r


def fit_edcircle(x, y):
    x = np.array(x)
    y = np.array(y)
    A = np.column_stack((x, y, np.ones_like(x)))
    b = -(x**2 + y**2)
    c = np.linalg.lstsq(A, b, rcond=None)[0]
    D, E, F = c
    x0 = -D / 2
    y0 = -E / 2
    r = np.sqrt((D**2 + E**2) / 4 - F)
    return x0, y0, r


def plot_r_theta_arrays(x, y):
    x = np.asarray(x).flatten()
    y = np.asarray(y).flatten()
    xc = np.mean(x)
    yc = np.mean(y)
    theta = np.arctan2(y - yc, x - xc)
    r = np.sqrt((x - xc)**2 + (y - yc)**2)
    idx = np.argsort(theta)
    return theta[idx], r[idx]


def run_experiment(params_json):
    params = json.loads(params_json)

    xc = 500
    yc = 500
    a = float(params['a'])
    b = float(params['b'])
    n_points = int(params['n_points'])
    sigma = float(params['sigma'])
    selected_outlier = params['selected_outlier']
    selected_fitting = params['selected_fitting']

    cluster_outliers = int(n_points * 0.02)
    near_ellipse_outliers = int(n_points * 0.02)
    random_outliers = 0
    random_seed = 42

    (
        x_in, y_in,
        x_out_cluster, y_out_cluster,
        x_out_near, y_out_near,
        x_out_random, y_out_random,
        X, Y, labels
    ) = generate_synthetic_ellipse(
        xc=xc,
        yc=yc,
        a=a,
        b=b,
        n_points=n_points,
        sigma=sigma,
        cluster_outliers=cluster_outliers,
        near_ellipse_outliers=near_ellipse_outliers,
        random_outliers=random_outliers,
        random_seed=random_seed
    )

    r_ref = (a + b) / 2.0

    outlier_methods = {
        "None": lambda x, y: (x, y),
        "Proposed Local Z-Score": lambda x, y: remove_outliers_local_zscore_proposed(x, y, threshold=3, window_size=60, std_window=60),
        "Z-Score": lambda x, y: remove_outliers_zscore(x, y, threshold=3.0),
        "MAD": lambda x, y: remove_outliers_mad(x, y, threshold=3),
        "DBSCAN": lambda x, y: remove_outliers_dbscan(x, y, eps=0.3, min_samples=5),
        "LOF": lambda x, y: remove_outliers_lof(x, y, n_neighbors=20),
        "Percentile": lambda x, y: remove_outliers_percentile(x, y, lower=2.275, upper=97.725),
    }

    fitting_methods = {
        "Geometric LS": fit_geometric_ls,
        "Pratt": fit_pratt,
        "Taubin": fit_taubin,
        "RANSAC": fit_ransac,
        "IRLS": fit_irls,
        "Hyper LS": fit_hyper_ls,
        "M-Estimator": fit_m_estimator,
        "LMedS": fit_lmeds,
        "TLS": fit_tls,
        "Bayesian": fit_bayesian,
        "Gradient Descent": fit_gradient_descent,
        "EDCircle": fit_edcircle,
    }

    all_results = []

    for out_name, out_func in outlier_methods.items():
        try:
            x_clean, y_clean = out_func(X, Y)
        except Exception:
            continue

        removed_count = len(X) - len(x_clean)

        if len(x_clean) < 3:
            continue

        for fit_name, fit_func in fitting_methods.items():
            try:
                x0, y0, r = fit_func(x_clean, y_clean)
                if not (np.isfinite(x0) and np.isfinite(y0) and np.isfinite(r)):
                    continue
                center_error = np.sqrt((x0 - xc) ** 2 + (y0 - yc) ** 2)
                radius_error = abs(r - r_ref)
                all_results.append({
                    "outlier_method": out_name,
                    "fitting_method": fit_name,
                    "n_remaining": int(len(x_clean)),
                    "n_removed": int(removed_count),
                    "x0": float(x0),
                    "y0": float(y0),
                    "r": float(r),
                    "center_error": float(center_error),
                    "radius_error": float(radius_error)
                })
            except Exception:
                continue

    all_results_sorted = sorted(all_results, key=lambda d: (d["radius_error"], d["center_error"]))

    x_sel, y_sel = outlier_methods[selected_outlier](X, Y)
    x0_sel, y0_sel, r_sel = fitting_methods[selected_fitting](x_sel, y_sel)
    center_error_sel = float(np.sqrt((x0_sel - xc) ** 2 + (y0_sel - yc) ** 2))
    radius_error_sel = float(abs(r_sel - r_ref))

    theta_orig, r_orig = plot_r_theta_arrays(X, Y)
    theta_filt, r_filt = plot_r_theta_arrays(x_sel, y_sel)

    th = np.linspace(0, 2 * np.pi, 400)
    x_circle_ref = xc + r_ref * np.cos(th)
    y_circle_ref = yc + r_ref * np.sin(th)
    x_circle_fit = x0_sel + r_sel * np.cos(th)
    y_circle_fit = y0_sel + r_sel * np.sin(th)

    result = {
        "params": {
            "sigma": sigma,
            "n_points": n_points,
            "a": a,
            "b": b,
            "cluster_outliers": int(cluster_outliers),
            "near_ellipse_outliers": int(near_ellipse_outliers),
            "random_outliers": int(random_outliers),
            "random_seed": int(random_seed)
        },
        "counts": {
            "inlier": int(len(x_in)),
            "cluster_outlier": int(len(x_out_cluster)),
            "near_outlier": int(len(x_out_near)),
            "random_outlier": int(len(x_out_random)),
            "total": int(len(X)),
            "remaining": int(len(x_sel)),
            "removed": int(len(X) - len(x_sel))
        },
        "scatter": {
            "x_in": x_in.tolist(),
            "y_in": y_in.tolist(),
            "x_out_cluster": x_out_cluster.tolist(),
            "y_out_cluster": y_out_cluster.tolist(),
            "x_out_near": x_out_near.tolist(),
            "y_out_near": y_out_near.tolist(),
            "x_out_random": x_out_random.tolist(),
            "y_out_random": y_out_random.tolist(),
            "x_filtered": np.asarray(x_sel).tolist(),
            "y_filtered": np.asarray(y_sel).tolist(),
            "x_circle_ref": x_circle_ref.tolist(),
            "y_circle_ref": y_circle_ref.tolist(),
            "x_circle_fit": x_circle_fit.tolist(),
            "y_circle_fit": y_circle_fit.tolist(),
            "x_true_center": [float(xc)],
            "y_true_center": [float(yc)],
            "x_est_center": [float(x0_sel)],
            "y_est_center": [float(y0_sel)]
        },
        "rtheta": {
            "theta_original": theta_orig.tolist(),
            "r_original": r_orig.tolist(),
            "theta_filtered": theta_filt.tolist(),
            "r_filtered": r_filt.tolist()
        },
        "selected_result": {
            "outlier_method": selected_outlier,
            "fitting_method": selected_fitting,
            "x0": float(x0_sel),
            "y0": float(y0_sel),
            "r": float(r_sel),
            "r_ref": float(r_ref),
            "center_error": center_error_sel,
            "radius_error": radius_error_sel
        },
        "top10": all_results_sorted[:10]
    }

    return json.dumps(result)
`;

function setStatus(msg) {
  document.getElementById('status_box').textContent = msg;
}

function updateSliderValues() {
  document.getElementById('sigma_val').textContent = document.getElementById('sigma').value;
  document.getElementById('n_points_val').textContent = document.getElementById('n_points').value;
  document.getElementById('a_val').textContent = document.getElementById('a').value;
  document.getElementById('b_val').textContent = document.getElementById('b').value;
}

async function initPy() {
  setStatus('Loading Pyodide core...');
  pyodide = await loadPyodide();

  setStatus('Loading numpy...');
  await pyodide.loadPackage(['numpy']);

  setStatus('Loading scipy...');
  await pyodide.loadPackage(['scipy']);

  setStatus('Loading pandas...');
  await pyodide.loadPackage(['pandas']);

  setStatus('Loading scikit-learn (this is slow)...');
  await pyodide.loadPackage(['scikit-learn']);

  setStatus('Initializing Python code...');
  await pyodide.runPythonAsync(pythonSource);

  pyReady = true;
  setStatus('Ready.');

  await runDemo();
}

function currentParams() {
  return {
    sigma: parseFloat(document.getElementById('sigma').value),
    n_points: parseInt(document.getElementById('n_points').value, 10),
    a: parseFloat(document.getElementById('a').value),
    b: parseFloat(document.getElementById('b').value),
    selected_outlier: document.getElementById('selected_outlier').value,
    selected_fitting: document.getElementById('selected_fitting').value
  };
}

function renderScatter(scatter) {
  const traces = [
    {
      x: scatter.x_in,
      y: scatter.y_in,
      mode: 'markers',
      type: 'scatter',
      name: 'Inlier',
      marker: { size: 5 }
    },
    {
      x: scatter.x_out_cluster,
      y: scatter.y_out_cluster,
      mode: 'markers',
      type: 'scatter',
      name: 'Cluster Outlier',
      marker: { size: 7, symbol: 'circle' }
    },
    {
      x: scatter.x_out_near,
      y: scatter.y_out_near,
      mode: 'markers',
      type: 'scatter',
      name: 'Near-Ellipse Outlier',
      marker: { size: 7, symbol: 'diamond' }
    },
    {
      x: scatter.x_out_random,
      y: scatter.y_out_random,
      mode: 'markers',
      type: 'scatter',
      name: 'Random Outlier',
      marker: { size: 7, symbol: 'x' }
    },
    {
      x: scatter.x_filtered,
      y: scatter.y_filtered,
      mode: 'markers',
      type: 'scatter',
      name: 'Filtered',
      marker: { size: 4, opacity: 0.65 }
    },
    {
      x: scatter.x_circle_ref,
      y: scatter.y_circle_ref,
      mode: 'lines',
      type: 'scatter',
      name: 'Reference Circle',
      line: { width: 2 }
    },
    {
      x: scatter.x_circle_fit,
      y: scatter.y_circle_fit,
      mode: 'lines',
      type: 'scatter',
      name: 'Fitted Circle',
      line: { width: 3 }
    },
    {
      x: scatter.x_true_center,
      y: scatter.y_true_center,
      mode: 'markers',
      type: 'scatter',
      name: 'True Center',
      marker: { size: 11, symbol: 'cross' }
    },
    {
      x: scatter.x_est_center,
      y: scatter.y_est_center,
      mode: 'markers',
      type: 'scatter',
      name: 'Estimated Center',
      marker: { size: 10, symbol: 'square' }
    }
  ];

  Plotly.newPlot('plot_scatter', traces, {
    title: 'Synthetic Ellipse, Outliers, Filtering, and Fitted Circle',
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
      marker: { size: 5 }
    },
    {
      x: rtheta.theta_filtered,
      y: rtheta.r_filtered,
      mode: 'markers',
      type: 'scatter',
      name: 'Filtered',
      marker: { size: 5 }
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
  const c = data.counts;
  const r = data.selected_result;
  document.getElementById('result_box').innerHTML = `
    <strong>Selected Combination</strong><br><br>
    <strong>Outlier Removal:</strong> ${r.outlier_method}<br>
    <strong>Fitting Method:</strong> ${r.fitting_method}<br><br>

    <strong>Inlier count:</strong> ${c.inlier}<br>
    <strong>Cluster outliers:</strong> ${c.cluster_outlier}<br>
    <strong>Near-ellipse outliers:</strong> ${c.near_outlier}<br>
    <strong>Random outliers:</strong> ${c.random_outlier}<br>
    <strong>Total points:</strong> ${c.total}<br>
    <strong>Remaining after filtering:</strong> ${c.remaining}<br>
    <strong>Removed:</strong> ${c.removed}<br><br>

    <strong>Estimated xc:</strong> ${r.x0.toFixed(4)}<br>
    <strong>Estimated yc:</strong> ${r.y0.toFixed(4)}<br>
    <strong>Estimated r:</strong> ${r.r.toFixed(4)}<br>
    <strong>Reference r:</strong> ${r.r_ref.toFixed(4)}<br>
    <strong>Center error:</strong> ${r.center_error.toFixed(4)}<br>
    <strong>Radius error:</strong> ${r.radius_error.toFixed(4)}
  `;
}

function renderTable(rows) {
  const tbody = document.querySelector('#results_table tbody');
  tbody.innerHTML = '';
  rows.forEach(row => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${row.outlier_method}</td>
      <td>${row.fitting_method}</td>
      <td>${row.n_remaining}</td>
      <td>${row.n_removed}</td>
      <td>${row.x0.toFixed(4)}</td>
      <td>${row.y0.toFixed(4)}</td>
      <td>${row.r.toFixed(4)}</td>
      <td>${row.center_error.toFixed(4)}</td>
      <td>${row.radius_error.toFixed(4)}</td>
    `;
    tbody.appendChild(tr);
  });
}

async function runDemo() {
  updateSliderValues();
  if (!pyReady) return;

  try {
    setStatus('Running analysis...');
    const params = currentParams();
    pyodide.globals.set('js_params', JSON.stringify(params));
    const resultJson = await pyodide.runPythonAsync('run_experiment(js_params)');
    const data = JSON.parse(resultJson);
    renderScatter(data.scatter);
    renderRTheta(data.rtheta);
    renderResultsBox(data);
    renderTable(data.top10);
    setStatus('Completed.');
  } catch (err) {
    console.error(err);
    setStatus('Error: ' + err.message);
  }
}

document.getElementById('run_btn').addEventListener('click', runDemo);
document.querySelectorAll('input, select').forEach(el => {
  el.addEventListener('input', updateSliderValues);
});

updateSliderValues();
initPy();
</script>
