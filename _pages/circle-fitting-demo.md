---
layout: single
title: "Interactive Circle Fitting Demo"
permalink: /circle-fitting-demo/
---

<h2>Interactive Synthetic Circle / Ellipse Generator</h2>

<label>Sigma:</label>
<input type="range" id="sigma" min="0" max="5" step="0.1" value="0.8">
<span id="sigma_val">0.8</span>
<br>

<label>Number of Points:</label>
<input type="range" id="n_points" min="100" max="2000" step="50" value="1000">
<span id="n_points_val">1000</span>
<br>

<label>a (radius x):</label>
<input type="range" id="a" min="100" max="1000" step="10" value="675">
<span id="a_val">675</span>
<br>

<label>b (radius y):</label>
<input type="range" id="b" min="100" max="1000" step="10" value="685">
<span id="b_val">685</span>
<br><br>

<div id="plot" style="width:800px;height:800px;"></div>

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<script>

function generateData() {
    let sigma = parseFloat(document.getElementById("sigma").value);
    let n_points = parseInt(document.getElementById("n_points").value);
    let a = parseFloat(document.getElementById("a").value);
    let b = parseFloat(document.getElementById("b").value);

    document.getElementById("sigma_val").innerText = sigma;
    document.getElementById("n_points_val").innerText = n_points;
    document.getElementById("a_val").innerText = a;
    document.getElementById("b_val").innerText = b;

    let xc = 500;
    let yc = 500;

    // 🔴 SABİT ORAN
    let cluster_outliers = Math.floor(n_points * 0.02);

    let x_in = [];
    let y_in = [];

    for (let i = 0; i < n_points; i++) {
        let theta = 2 * Math.PI * i / n_points;

        let x = xc + a * Math.cos(theta) + sigma * randn();
        let y = yc + b * Math.sin(theta) + sigma * randn();

        x_in.push(x);
        y_in.push(y);
    }

    // 🔴 Cluster outliers
    let x_out = [];
    let y_out = [];

    for (let i = 0; i < cluster_outliers; i++) {
        x_out.push(xc + a + 20 + randn()*5);
        y_out.push(yc + randn()*5);
    }

    return {x_in, y_in, x_out, y_out};
}

// Gaussian random
function randn() {
    let u = 0, v = 0;
    while(u === 0) u = Math.random();
    while(v === 0) v = Math.random();
    return Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);
}

function plot() {
    let data = generateData();

    let trace1 = {
        x: data.x_in,
        y: data.y_in,
        mode: 'markers',
        type: 'scatter',
        name: 'Inliers'
    };

    let trace2 = {
        x: data.x_out,
        y: data.y_out,
        mode: 'markers',
        type: 'scatter',
        name: 'Cluster Outliers',
        marker: { size: 8 }
    };

    let layout = {
        title: 'Synthetic Data',
        xaxis: {scaleanchor: "y"},
        yaxis: {},
    };

    Plotly.newPlot('plot', [trace1, trace2], layout);
}

document.querySelectorAll("input").forEach(el => {
    el.addEventListener("input", plot);
});

plot();

</script>
