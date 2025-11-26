<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>House Price Prediction - Demo</title>
  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width: 900px; margin: 32px auto; padding: 0 16px; color: #111; }
    h1 { margin-top: 0; }
    .card { border: 1px solid #e6e6e6; padding: 16px; border-radius: 8px; margin-bottom: 16px; background: #fafafa; }
    label { display:block; margin:8px 0 4px; font-weight:600; }
    input[type="number"], input[type="text"], select { width:100%; padding:8px; border-radius:6px; border:1px solid #ccc; box-sizing:border-box; }
    .row { display:flex; gap:12px; flex-wrap:wrap; }
    .col { flex:1 1 200px; min-width:180px; }
    button { padding:10px 14px; border-radius:6px; border:0; background:#0066cc; color:#fff; cursor:pointer; }
    button.secondary { background:#444; }
    pre { background:#111; color:#f6f6f6; padding:12px; border-radius:6px; overflow:auto; }
    .result { margin-top:12px; font-size:1.1rem; font-weight:700; }
    .note { color:#555; font-size:0.95rem; }
  </style>
</head>
<body>
  <h1>House Price Prediction — Demo UI</h1>
  <p class="note">This page is a simple front-end that sends input to a prediction API. It does not include the model itself. Use the Python backend (or any server) to accept the requests described below and return JSON responses.</p>

  <div class="card">
    <h2>Single-sample prediction</h2>
    <p class="note">Enter the 8 features in the same order used by the California Housing dataset.</p>

    <form id="sample-form" onsubmit="event.preventDefault(); predictSample();">
      <div class="row">
        <div class="col">
          <label for="medinc">MedInc (median income)</label>
          <input id="medinc" type="number" step="any" value="8.325" />
        </div>
        <div class="col">
          <label for="houseage">HouseAge</label>
          <input id="houseage" type="number" step="any" value="41" />
        </div>
        <div class="col">
          <label for="averooms">AveRooms</label>
          <input id="averooms" type="number" step="any" value="6.984127" />
        </div>
        <div class="col">
          <label for="avebedrms">AveBedrms</label>
          <input id="avebedrms" type="number" step="any" value="1.02381" />
        </div>
      </div>

      <div class="row" style="margin-top:8px;">
        <div class="col">
          <label for="population">Population</label>
          <input id="population" type="number" step="any" value="322" />
        </div>
        <div class="col">
          <label for="aveoccup">AveOccup</label>
          <input id="aveoccup" type="number" step="any" value="2.555556" />
        </div>
        <div class="col">
          <label for="latitude">Latitude</label>
          <input id="latitude" type="number" step="any" value="37.88" />
        </div>
        <div class="col">
          <label for="longitude">Longitude</label>
          <input id="longitude" type="number" step="any" value="-122.23" />
        </div>
      </div>

      <div style="margin-top:12px;">
        <button type="submit">Predict</button>
        <button type="button" class="secondary" onclick="fillExample()">Fill example</button>
      </div>
    </form>

    <div id="sample-result" class="result" aria-live="polite"></div>
  </div>

  <div class="card">
    <h2>CSV batch prediction</h2>
    <p class="note">Upload a CSV with a header row matching these columns (order can be different if header names match):</p>
    <pre>MedInc,HouseAge,AveRooms,AveBedrms,Population,AveOccup,Latitude,Longitude
8.325,41,6.984127,1.023810,322,2.555556,37.88,-122.23
...</pre>

    <form id="csv-form" onsubmit="event.preventDefault(); predictCSV();">
      <label for="csv-file">Select CSV file</label>
      <input id="csv-file" type="file" accept=".csv" />
      <div style="margin-top:12px;">
        <button type="submit">Upload & Predict</button>
      </div>
    </form>

    <div id="csv-result" class="result" aria-live="polite"></div>
    <div id="csv-table" style="margin-top:12px;"></div>
  </div>

  <div class="card">
    <h2>API contract (what the frontend sends)</h2>
    <p class="note">Implement these endpoints on your backend to receive requests from this page.</p>

    <h4>POST /api/predict</h4>
    <pre>{
  "values": [MedInc, HouseAge, AveRooms, AveBedrms, Population, AveOccup, Latitude, Longitude]
}</pre>
    <p class="note">Response (example):</p>
    <pre>{
  "predicted": 3.45
}</pre>

    <h4>POST /api/predict_csv</h4>
    <pre>Form-data:
  file: &lt;CSV file&gt;
</pre>
    <p class="note">Response (example):</p>
    <pre>{
  "predictions": [3.45, 2.22, 4.11],
  "rows": 3
}</pre>

    <p class="note">If you prefer to implement a single endpoint, adapt the fetch URLs in the JS below.</p>
  </div>

  <script>
    // Helper: update an element's text
    function setText(id, text) {
      document.getElementById(id).textContent = text;
    }

    function fillExample() {
      document.getElementById('medinc').value = "8.325";
      document.getElementById('houseage').value = "41";
      document.getElementById('averooms').value = "6.984127";
      document.getElementById('avebedrms').value = "1.02381";
      document.getElementById('population').value = "322";
      document.getElementById('aveoccup').value = "2.555556";
      document.getElementById('latitude').value = "37.88";
      document.getElementById('longitude').value = "-122.23";
      setText('sample-result', '');
    }

    async function predictSample() {
      setText('sample-result', 'Predicting…');

      const values = [
        parseFloat(document.getElementById('medinc').value),
        parseFloat(document.getElementById('houseage').value),
        parseFloat(document.getElementById('averooms').value),
        parseFloat(document.getElementById('avebedrms').value),
        parseFloat(document.getElementById('population').value),
        parseFloat(document.getElementById('aveoccup').value),
        parseFloat(document.getElementById('latitude').value),
        parseFloat(document.getElementById('longitude').value)
      ];

      // Basic client-side validation
      if (values.some(v => Number.isNaN(v))) {
        setText('sample-result', 'Please enter valid numeric values for all fields.');
        return;
      }

      try {
        // POST to your backend prediction endpoint.
        // Change the URL if your server uses a different path/port (e.g. http://localhost:8000/api/predict)
        const res = await fetch('/api/predict', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ values })
        });

        if (!res.ok) {
          const txt = await res.text();
          setText('sample-result', `Server error: ${res.status} ${res.statusText} — ${txt}`);
          return;
        }

        const data = await res.json();
        // Expecting { predicted: <number> } or { predictions: [<number>] }
        let predicted;
        if (Array.isArray(data.predictions)) predicted = data.predictions[0];
        else predicted = data.predicted ?? data.value ?? null;

        if (predicted === null || predicted === undefined) {
          setText('sample-result', 'Unexpected server response: ' + JSON.stringify(data));
        } else {
          setText('sample-result', 'Predicted median house value: ' + Number(predicted).toFixed(3));
        }
      } catch (err) {
        setText('sample-result', 'Network error: ' + err.message);
      }
    }

    async function predictCSV() {
      setText('csv-result', 'Uploading & predicting…');
      document.getElementById('csv-table').innerHTML = '';

      const fileInput = document.getElementById('csv-file');
      if (!fileInput.files || fileInput.files.length === 0) {
        setText('csv-result', 'Please choose a CSV file first.');
        return;
      }

      const form = new FormData();
      form.append('file', fileInput.files[0]);

      try {
        // POST to your backend CSV prediction endpoint.
        // Change URL if your server uses a different path/port.
        const res = await fetch('/api/predict_csv', {
          method: 'POST',
          body: form
        });

        if (!res.ok) {
          const txt = await res.text();
          setText('csv-result', `Server error: ${res.status} ${res.statusText} — ${txt}`);
          return;
        }

        const data = await res.json();
        // Expecting { predictions: [...], rows: n } or similar.
        if (!Array.isArray(data.predictions)) {
          setText('csv-result', 'Unexpected server response: ' + JSON.stringify(data));
          return;
        }

        setText('csv-result', `Predicted ${data.predictions.length} rows.`);
        // Show small table of predictions
        const table = document.createElement('table');
        table.style.width = '100%';
        table.style.borderCollapse = 'collapse';
        const thead = document.createElement('thead');
        thead.innerHTML = '<tr><th style="text-align:left; padding:8px; border-bottom:1px solid #ddd">Row</th><th style="text-align:right; padding:8px; border-bottom:1px solid #ddd">Prediction</th></tr>';
        table.appendChild(thead);
        const tbody = document.createElement('tbody');
        data.predictions.forEach((p, i) => {
          const tr = document.createElement('tr');
          tr.innerHTML = `<td style="padding:8px; border-bottom:1px solid #f0f0f0">${i+1}</td><td style="padding:8px; text-align:right; border-bottom:1px solid #f0f0f0">${Number(p).toFixed(3)}</td>`;
          tbody.appendChild(tr);
        });
        table.appendChild(tbody);
        document.getElementById('csv-table').appendChild(table);

      } catch (err) {
        setText('csv-result', 'Network error: ' + err.message);
      }
    }

    // Optional: allow CORS testing against localhost servers on a different port
    // If you run your backend on another origin, replace fetch URLs with full origin, e.g.:
    // const API_BASE = 'http://localhost:8000';
    // fetch(API_BASE + '/api/predict', ...)
  </script>
</body>
</html>

