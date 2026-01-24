<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Income vs Expense Prototype (MoM + Recurring Memory)</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <style>
    :root {
      --bg: #0b1220;
      --card: #111a2e;
      --text: #e6eefc;
      --muted: #9bb0d1;
      --line: #21304f;
      --accent: #6aa8ff;
      --danger: #ff6a7a;
      --ok: #5cffb3;
      --warning: #ffd36a;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: radial-gradient(1200px 800px at 20% 0%, #162447 0%, var(--bg) 55%);
      color: var(--text);
      padding: 20px;
    }
    .wrap { max-width: 1100px; margin: 0 auto; }
    header { display: flex; align-items: flex-start; justify-content: space-between; gap: 12px; margin-bottom: 16px; }
    h1 { font-size: 18px; margin: 0; letter-spacing: .3px; }
    .sub { color: var(--muted); font-size: 13px; margin-top: 4px; line-height: 1.4; }
    .grid {
      display: grid;
      grid-template-columns: 380px 1fr;
      gap: 14px;
    }
    .card {
      background: linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border: 1px solid rgba(255,255,255,.07);
      border-radius: 14px;
      padding: 14px;
      box-shadow: 0 18px 45px rgba(0,0,0,.35);
    }
    .card h2 { font-size: 14px; margin: 0 0 10px 0; color: #d8e6ff; }
    label { display:block; font-size: 12px; color: var(--muted); margin-bottom: 6px; }
    input, select, button {
      width: 100%;
      background: rgba(255,255,255,.04);
      border: 1px solid rgba(255,255,255,.10);
      color: var(--text);
      border-radius: 10px;
      padding: 10px 10px;
      outline: none;
    }
    input[type="checkbox"] { width: auto; transform: translateY(1px); }
    .row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .row3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; }
    .mt8 { margin-top: 8px; }
    .mt10 { margin-top: 10px; }
    .mt12 { margin-top: 12px; }
    .btnrow { display:flex; gap:10px; margin-top: 10px; }
    button { cursor: pointer; font-weight: 600; }
    button.primary { background: rgba(106,168,255,.18); border-color: rgba(106,168,255,.35); }
    button.danger  { background: rgba(255,106,122,.14); border-color: rgba(255,106,122,.30); }
    button.ghost   { background: transparent; }
    .hint { font-size: 12px; color: var(--muted); margin-top: 8px; line-height: 1.35; }
    .kpis { display:grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 10px; }
    .kpi {
      background: rgba(0,0,0,.22);
      border: 1px solid rgba(255,255,255,.06);
      border-radius: 12px;
      padding: 10px;
    }
    .kpi .label { font-size: 11px; color: var(--muted); }
    .kpi .value { font-size: 16px; margin-top: 2px; font-weight: 700; }
    .value.ok { color: var(--ok); }
    .value.bad { color: var(--danger); }
    .value.warn { color: var(--warning); }
    table { width:100%; border-collapse: collapse; margin-top: 10px; }
    th, td { font-size: 12px; padding: 9px 8px; border-bottom: 1px solid rgba(255,255,255,.08); }
    th { text-align:left; color: var(--muted); font-weight: 600; }
    td.right { text-align:right; }
    .tag {
      display:inline-flex; align-items:center; gap: 6px;
      padding: 2px 8px; border-radius: 999px; font-size: 11px;
      border: 1px solid rgba(255,255,255,.10); color: var(--text);
      background: rgba(255,255,255,.04);
    }
    .tag.income { border-color: rgba(92,255,179,.35); background: rgba(92,255,179,.12); }
    .tag.expense { border-color: rgba(255,106,122,.35); background: rgba(255,106,122,.12); }
    .tag.recurring { border-color: rgba(255,211,106,.35); background: rgba(255,211,106,.12); color: #fff3d1; }
    .flex { display:flex; align-items:center; gap:10px; flex-wrap: wrap; }
    .small { font-size: 12px; color: var(--muted); }
    .chartWrap { height: 340px; }
    @media (max-width: 980px) {
      .grid { grid-template-columns: 1fr; }
      .chartWrap { height: 320px; }
    }
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Projected Income vs Projected Expense (MoM)</h1>
        <div class="sub">
          Prototype demonstrates: month-over-month history, recurring checkboxes, and persistent memory via localStorage.
        </div>
      </div>
      <div class="small" id="status"></div>
    </header>

    <div class="grid">
      <!-- LEFT: inputs & recurring items -->
      <section class="card">
        <h2>Inputs</h2>

        <div class="row">
          <div>
            <label for="startMonth">Start month</label>
            <input id="startMonth" type="month" />
          </div>
          <div>
            <label for="monthsCount">Months to display</label>
            <select id="monthsCount">
              <option value="6">6</option>
              <option value="12" selected>12</option>
              <option value="18">18</option>
              <option value="24">24</option>
            </select>
          </div>
        </div>

        <div class="mt12">
          <h2>Add a projected item</h2>
          <div class="row">
            <div>
              <label for="type">Type</label>
              <select id="type">
                <option value="income">Income</option>
                <option value="expense">Expense</option>
              </select>
            </div>
            <div>
              <label for="name">Name</label>
              <input id="name" placeholder="e.g., Paycheck, Rent, Utilities" />
            </div>
          </div>

          <div class="row mt10">
            <div>
              <label for="amount">Amount (monthly)</label>
              <input id="amount" type="number" step="0.01" placeholder="e.g., 3500" />
            </div>
            <div>
              <label for="monthApplies">Month applies</label>
              <input id="monthApplies" type="month" />
            </div>
          </div>

          <div class="flex mt10">
            <label class="flex" style="margin:0;">
              <input id="recurring" type="checkbox" />
              <span class="small">Recurring (store in memory and always factor in)</span>
            </label>
          </div>

          <div class="btnrow">
            <button class="primary" id="addBtn">Add item</button>
            <button class="ghost" id="demoBtn">Load demo data</button>
          </div>

          <div class="hint">
            Recurring items persist across sessions and will be applied to every month in the chart window (from start month forward).
            Non-recurring items apply only to the selected month.
          </div>

          <div class="btnrow">
            <button class="danger" id="clearBtn">Clear all saved data</button>
          </div>
        </div>

        <div class="mt12">
          <h2>Saved items (memory)</h2>
          <div class="small">Recurring items automatically roll forward month over month.</div>
          <div style="max-height: 240px; overflow:auto; margin-top: 8px;">
            <table>
              <thead>
                <tr>
                  <th>Item</th>
                  <th>Type</th>
                  <th class="right">Amount</th>
                  <th></th>
                </tr>
              </thead>
              <tbody id="itemsTbody"></tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- RIGHT: chart & MoM table -->
      <section class="card">
        <h2>Month-over-month projection</h2>

        <div class="kpis">
          <div class="kpi">
            <div class="label">Avg Monthly Income</div>
            <div class="value" id="kpiIncome">$0</div>
          </div>
          <div class="kpi">
            <div class="label">Avg Monthly Expense</div>
            <div class="value" id="kpiExpense">$0</div>
          </div>
          <div class="kpi">
            <div class="label">Avg Net (Income - Expense)</div>
            <div class="value" id="kpiNet">$0</div>
          </div>
        </div>

        <div class="chartWrap mt10">
          <canvas id="lineChart"></canvas>
        </div>

        <div class="mt12">
          <h2>History (month-by-month)</h2>
          <table>
            <thead>
              <tr>
                <th>Month</th>
                <th class="right">Projected Income</th>
                <th class="right">Projected Expense</th>
                <th class="right">Net</th>
              </tr>
            </thead>
            <tbody id="momTbody"></tbody>
          </table>
        </div>
      </section>
    </div>
  </div>

  <script>
    // -----------------------------
    // Persistence (localStorage)
    // -----------------------------
    const STORAGE_KEY = "incomeExpensePrototype.v1";

    function loadState() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (!raw) return { items: [], startMonth: null, monthsCount: 12 };
        const parsed = JSON.parse(raw);
        return {
          items: Array.isArray(parsed.items) ? parsed.items : [],
          startMonth: parsed.startMonth || null,
          monthsCount: Number(parsed.monthsCount || 12)
        };
      } catch {
        return { items: [], startMonth: null, monthsCount: 12 };
      }
    }

    function saveState(state) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
      setStatus("Saved.");
    }

    function setStatus(msg) {
      const el = document.getElementById("status");
      el.textContent = msg;
      clearTimeout(setStatus._t);
      setStatus._t = setTimeout(() => el.textContent = "", 1200);
    }

    // -----------------------------
    // Date helpers
    // -----------------------------
    function todayMonthValue() {
      const d = new Date();
      const y = d.getFullYear();
      const m = String(d.getMonth() + 1).padStart(2, "0");
      return `${y}-${m}`;
    }

    function addMonths(ym, add) {
      // ym: "YYYY-MM"
      const [Y, M] = ym.split("-").map(Number);
      const d = new Date(Y, M - 1, 1);
      d.setMonth(d.getMonth() + add);
      const y = d.getFullYear();
      const m = String(d.getMonth() + 1).padStart(2, "0");
      return `${y}-${m}`;
    }

    function formatMonthLabel(ym) {
      const [Y, M] = ym.split("-").map(Number);
      const d = new Date(Y, M - 1, 1);
      return d.toLocaleString(undefined, { month: "short", year: "numeric" });
    }

    function currency(n) {
      return new Intl.NumberFormat(undefined, { style: "currency", currency: "USD" }).format(n);
    }

    // -----------------------------
    // Projection logic
    // Items shape:
    // { id, type: "income"|"expense", name, amount, monthApplies, recurring: boolean }
    // - recurring: applies to every month >= monthApplies (in this simple prototype)
    // - non-recurring: applies only to monthApplies
    // -----------------------------
    function computeSeries(state, startMonth, monthsCount) {
      const months = [];
      for (let i = 0; i < monthsCount; i++) months.push(addMonths(startMonth, i));

      const income = [];
      const expense = [];
      const net = [];

      for (const ym of months) {
        let inc = 0;
        let exp = 0;

        for (const it of state.items) {
          if (!it || !it.type || !it.amount) continue;

          const amount = Number(it.amount) || 0;
          const appliesMonth = it.monthApplies;

          const isApplicable =
            it.recurring
              ? (appliesMonth && ym >= appliesMonth) // rolls forward from its start
              : (appliesMonth === ym);

          if (!isApplicable) continue;

          if (it.type === "income") inc += amount;
          if (it.type === "expense") exp += amount;
        }

        income.push(inc);
        expense.push(exp);
        net.push(inc - exp);
      }

      return { months, income, expense, net };
    }

    // -----------------------------
    // Chart
    // -----------------------------
    let chart;

    function renderChart(series) {
      const ctx = document.getElementById("lineChart");
      const labels = series.months.map(formatMonthLabel);

      const data = {
        labels,
        datasets: [
          {
            label: "Projected Income",
            data: series.income,
            tension: 0.25,
            borderWidth: 2,
            pointRadius: 2
          },
          {
            label: "Projected Expense",
            data: series.expense,
            tension: 0.25,
            borderWidth: 2,
            pointRadius: 2
          },
          {
            // "Difference" visualization as a net line (optional but helpful)
            label: "Net (Income - Expense)",
            data: series.net,
            tension: 0.25,
            borderWidth: 2,
            pointRadius: 2,
            borderDash: [6, 6]
          }
        ]
      };

      const options = {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { labels: { color: "#e6eefc" } },
          tooltip: {
            callbacks: {
              label: (ctx) => `${ctx.dataset.label}: ${currency(ctx.parsed.y)}`
            }
          }
        },
        scales: {
          x: { ticks: { color: "#9bb0d1" }, grid: { color: "rgba(255,255,255,.06)" } },
          y: { ticks: { color: "#9bb0d1" }, grid: { color: "rgba(255,255,255,.06)" } }
        }
      };

      if (chart) chart.destroy();
      chart = new Chart(ctx, { type: "line", data, options });
    }

    // -----------------------------
    // UI rendering
    // -----------------------------
    function renderItemsTable(state) {
      const tbody = document.getElementById("itemsTbody");
      tbody.innerHTML = "";

      if (!state.items.length) {
        const tr = document.createElement("tr");
        tr.innerHTML = `<td colspan="4" class="small">No saved items yet.</td>`;
        tbody.appendChild(tr);
        return;
      }

      for (const it of state.items) {
        const tr = document.createElement("tr");
        const typeTag = it.type === "income"
          ? `<span class="tag income">Income</span>`
          : `<span class="tag expense">Expense</span>`;
        const recTag = it.recurring ? ` <span class="tag recurring">Recurring</span>` : "";
        tr.innerHTML = `
          <td>
            <div style="font-weight:700;">${escapeHtml(it.name || "(Unnamed)")}</div>
            <div class="small">${escapeHtml(it.monthApplies || "")}${recTag}</div>
          </td>
          <td>${typeTag}</td>
          <td class="right">${currency(Number(it.amount || 0))}</td>
          <td class="right"><button class="ghost" data-del="${it.id}">Remove</button></td>
        `;
        tbody.appendChild(tr);
      }

      tbody.querySelectorAll("button[data-del]").forEach(btn => {
        btn.addEventListener("click", () => {
          const id = btn.getAttribute("data-del");
          state.items = state.items.filter(x => x.id !== id);
          saveState(state);
          refresh(state);
        });
      });
    }

    function renderMoMTable(series) {
      const tbody = document.getElementById("momTbody");
      tbody.innerHTML = "";

      for (let i = 0; i < series.months.length; i++) {
        const ym = series.months[i];
        const inc = series.income[i];
        const exp = series.expense[i];
        const net = series.net[i];

        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${formatMonthLabel(ym)}</td>
          <td class="right">${currency(inc)}</td>
          <td class="right">${currency(exp)}</td>
          <td class="right" style="font-weight:700;">${currency(net)}</td>
        `;
        tbody.appendChild(tr);
      }

      // KPI rollups
      const avg = (arr) => arr.reduce((a,b)=>a+b,0) / (arr.length || 1);
      const avgInc = avg(series.income);
      const avgExp = avg(series.expense);
      const avgNet = avg(series.net);

      document.getElementById("kpiIncome").textContent = currency(avgInc);
      document.getElementById("kpiExpense").textContent = currency(avgExp);

      const netEl = document.getElementById("kpiNet");
      netEl.textContent = currency(avgNet);
      netEl.classList.remove("ok","bad","warn");
      netEl.classList.add(avgNet > 0 ? "ok" : (avgNet < 0 ? "bad" : "warn"));
    }

    function escapeHtml(s) {
      return String(s)
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;")
        .replaceAll('"', "&quot;")
        .replaceAll("'", "&#039;");
    }

    function refresh(state) {
      // pull current UI values
      const startMonth = document.getElementById("startMonth").value || todayMonthValue();
      const monthsCount = Number(document.getElementById("monthsCount").value || 12);

      // persist these preferences too
      state.startMonth = startMonth;
      state.monthsCount = monthsCount;
      saveState(state);

      const series = computeSeries(state, startMonth, monthsCount);

      renderItemsTable(state);
      renderChart(series);
      renderMoMTable(series);
    }

    // -----------------------------
    // Wire up
    // -----------------------------
    const state = loadState();

    // defaults
    document.getElementById("startMonth").value = state.startMonth || todayMonthValue();
    document.getElementById("monthsCount").value = String(state.monthsCount || 12);
    document.getElementById("monthApplies").value = (state.startMonth || todayMonthValue());

    document.getElementById("startMonth").addEventListener("change", () => refresh(state));
    document.getElementById("monthsCount").addEventListener("change", () => refresh(state));

    document.getElementById("addBtn").addEventListener("click", () => {
      const type = document.getElementById("type").value;
      const name = document.getElementById("name").value.trim();
      const amount = Number(document.getElementById("amount").value);
      const monthApplies = document.getElementById("monthApplies").value;
      const recurring = document.getElementById("recurring").checked;

      if (!name) return alert("Please enter a name.");
      if (!Number.isFinite(amount) || amount <= 0) return alert("Please enter a valid amount > 0.");
      if (!monthApplies) return alert("Please select a month.");

      state.items.push({
        id: crypto.randomUUID ? crypto.randomUUID() : String(Date.now() + Math.random()),
        type,
        name,
        amount,
        monthApplies,
        recurring
      });

      // Clear minimal fields for next entry
      document.getElementById("name").value = "";
      document.getElementById("amount").value = "";
      document.getElementById("recurring").checked = false;

      saveState(state);
      refresh(state);
    });

    document.getElementById("demoBtn").addEventListener("click", () => {
      const base = document.getElementById("startMonth").value || todayMonthValue();
      state.items = [
        { id: "d1", type: "income",  name: "Paycheck", amount: 5200, monthApplies: base, recurring: true },
        { id: "d2", type: "income",  name: "Side Income", amount: 400, monthApplies: base, recurring: true },
        { id: "d3", type: "expense", name: "Rent/Mortgage", amount: 2100, monthApplies: base, recurring: true },
        { id: "d4", type: "expense", name: "Utilities", amount: 280, monthApplies: base, recurring: true },
        { id: "d5", type: "expense", name: "Groceries", amount: 650, monthApplies: base, recurring: true },
        { id: "d6", type: "expense", name: "Car Repair (one-time)", amount: 900, monthApplies: addMonths(base, 2), recurring: false }
      ];
      saveState(state);
      refresh(state);
    });

    document.getElementById("clearBtn").addEventListener("click", () => {
      if (!confirm("Clear all saved data (items + preferences)?")) return;
      localStorage.removeItem(STORAGE_KEY);
      state.items = [];
      state.startMonth = null;
      state.monthsCount = 12;
      document.getElementById("startMonth").value = todayMonthValue();
      document.getElementById("monthsCount").value = "12";
      document.getElementById("monthApplies").value = todayMonthValue();
      setStatus("Cleared.");
      refresh(state);
    });

    // initial render
    refresh(state);
  </script>
</body>
</html>
