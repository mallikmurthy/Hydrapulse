<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>IoT Real-Time Water Potability & Work Allocation</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen p-4 sm:p-6 font-sans">

  <div class="max-w-4xl mx-auto space-y-6">

    <!-- Header -->
    <header class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
      <div>
        <h1 class="text-xl sm:text-2xl font-bold text-slate-900 flex items-center gap-2">
          <span>💧</span> Smart Water Potability & Usability Hub
        </h1>
        <p class="text-xs sm:text-sm text-slate-500 mt-1">Live telemetry analytics: TDS, pH, Turbidity, Temp & Color sensors</p>
      </div>
      <div id="livePill" class="flex items-center gap-2 px-3 py-1.5 rounded-full bg-emerald-50 text-emerald-700 border border-emerald-200 text-xs font-semibold">
        <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse"></span>
        ESP32 Connected
      </div>
    </header>

    <!-- Real-time Sensor Metric Gauges -->
    <div class="grid grid-cols-2 sm:grid-cols-5 gap-3">
      <!-- TDS -->
      <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm text-center">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">TDS</span>
        <div id="valTds" class="text-xl sm:text-2xl font-black text-slate-900 mt-1">--</div>
        <span class="text-[11px] text-slate-500">ppm</span>
      </div>

      <!-- pH -->
      <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm text-center">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">pH Level</span>
        <div id="valPh" class="text-xl sm:text-2xl font-black text-slate-900 mt-1">--</div>
        <span class="text-[11px] text-slate-500">6.5 - 8.5 target</span>
      </div>

      <!-- Turbidity -->
      <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm text-center">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">Turbidity</span>
        <div id="valTurb" class="text-xl sm:text-2xl font-black text-slate-900 mt-1">--</div>
        <span class="text-[11px] text-slate-500">NTU</span>
      </div>

      <!-- Temperature -->
      <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm text-center">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">Temp</span>
        <div id="valTemp" class="text-xl sm:text-2xl font-black text-slate-900 mt-1">--</div>
        <span class="text-[11px] text-slate-500">&deg;C</span>
      </div>

      <!-- Color Sensor RGB -->
      <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-sm text-center col-span-2 sm:col-span-1">
        <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">Water Color</span>
        <div class="flex items-center justify-center gap-2 mt-2">
          <div id="colorSwatch" class="w-6 h-6 rounded-full border border-slate-300 bg-blue-100 shadow-inner"></div>
          <span id="valRgb" class="text-xs font-medium text-slate-700">Clear</span>
        </div>
      </div>
    </div>

    <!-- Potability Status & Shelf-life Card -->
    <div id="statusCard" class="p-5 rounded-2xl border transition-all duration-300">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
        <div>
          <span class="text-[11px] font-bold uppercase tracking-wider opacity-75">Analysis Result</span>
          <h2 id="statusHeadline" class="text-xl font-bold">Evaluating sensors...</h2>
          <p id="statusAdvice" class="text-xs sm:text-sm mt-1"></p>
        </div>
        <div class="bg-white/90 px-4 py-2 rounded-xl border border-black/5 text-center min-w-[170px]">
          <span class="text-[10px] uppercase font-bold text-slate-500 block">Safe Drinking Duration</span>
          <span id="hoursLeftDisplay" class="text-lg font-black text-slate-900">-- Hours</span>
        </div>
      </div>
    </div>

    <!-- Works & Application Breakdown -->
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      
      <!-- Allowed Works -->
      <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
        <h3 class="text-xs font-bold uppercase tracking-wider text-emerald-700 flex items-center gap-2 mb-3">
          <span class="w-2 h-2 rounded-full bg-emerald-500"></span>
          Tasks You Can Use This Water For
        </h3>
        <div id="allowedList" class="space-y-2">
          <p class="text-xs text-slate-400">Awaiting sensor evaluation...</p>
        </div>
      </div>

      <!-- Restricted Works -->
      <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
        <h3 class="text-xs font-bold uppercase tracking-wider text-rose-700 flex items-center gap-2 mb-3">
          <span class="w-2 h-2 rounded-full bg-rose-500"></span>
          Tasks To Strictly Avoid
        </h3>
        <div id="restrictedList" class="space-y-2">
          <p class="text-xs text-slate-400">No restrictions detected.</p>
        </div>
      </div>

    </div>

  </div>

  <script>
    async function fetchTelemetry() {
      try {
        const res = await fetch("/api/water/live-analysis");
        const data = await res.json();
        
        // Populate Sensor Values
        document.getElementById("valTds").innerText = data.telemetry.tds.toFixed(0);
        document.getElementById("valPh").innerText = data.telemetry.ph.toFixed(1);
        document.getElementById("valTurb").innerText = data.telemetry.turbidity.toFixed(1);
        document.getElementById("valTemp").innerText = data.telemetry.temperature.toFixed(1);
        
        // Color Sensor swatch
        const r = data.telemetry.r, g = data.telemetry.g, b = data.telemetry.b;
        document.getElementById("colorSwatch").style.backgroundColor = `rgb(${r}, ${g}, ${b})`;
        document.getElementById("valRgb").innerText = `(${r},${g},${b})`;

        // Status Card Styling
        const card = document.getElementById("statusCard");
        const headline = document.getElementById("statusHeadline");
        const advice = document.getElementById("statusAdvice");
        const hours = document.getElementById("hoursLeftDisplay");

        headline.innerText = data.status_text;
        advice.innerText = data.advice;

        if (data.badge_style === "safe") {
          card.className = "p-5 rounded-2xl border bg-emerald-50 border-emerald-200 text-emerald-950";
          hours.innerText = `${data.drinking_hours_left} Hours`;
          hours.className = "text-lg font-black text-emerald-700";
        } else if (data.badge_style === "warning") {
          card.className = "p-5 rounded-2xl border bg-amber-50 border-amber-200 text-amber-950";
          hours.innerText = "0h (Boil First)";
          hours.className = "text-lg font-black text-amber-700";
        } else {
          card.className = "p-5 rounded-2xl border bg-rose-50 border-rose-200 text-rose-950";
          hours.innerText = "Unsafe (0h)";
          hours.className = "text-lg font-black text-rose-700";
        }

        // Render Allowed Works
        const allowedBox = document.getElementById("allowedList");
        allowedBox.innerHTML = data.allowed_works.map(item => `
          <div class="p-2.5 rounded-lg bg-emerald-50/70 border border-emerald-100 text-xs font-semibold text-emerald-900 flex items-center gap-2">
            <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span> ${item}
          </div>
        `).join("");

        // Render Restricted Works
        const restrictedBox = document.getElementById("restrictedList");
        if (data.restricted_works.length > 0) {
          restrictedBox.innerHTML = data.restricted_works.map(item => `
            <div class="p-2.5 rounded-lg bg-rose-50/70 border border-rose-100 text-xs font-semibold text-rose-900 flex items-center gap-2">
              <span class="w-1.5 h-1.5 rounded-full bg-rose-500"></span> ${item}
            </div>
          `).join("");
        } else {
          restrictedBox.innerHTML = `<div class="p-2.5 rounded-lg bg-slate-50 border border-slate-100 text-xs text-slate-500 font-medium">Water meets all potable thresholds. No current restrictions.</div>`;
        }

      } catch (err) {
        console.error("Telemetry poll failed:", err);
      }
    }

    // Poll live data every 3 seconds
    setInterval(fetchTelemetry, 3000);
    fetchTelemetry();
  </script>
</body>
</html>
