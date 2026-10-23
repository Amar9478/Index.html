<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>QC Register - Jyoti Gas</title>
  <style>
    * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
    body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: #f2f2f7; margin: 0; padding: 12px; }
    .card { background: #fff; border-radius: 12px; padding: 14px; margin-bottom: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
    h2 { font-size: 17px; text-align: center; margin: 0 0 2px 0; color: #000; }
    h3 { font-size: 13px; text-align: center; margin: 0 0 10px 0; color: #555; }
    label { font-size: 11px; font-weight: 700; color: #666; text-transform: uppercase; display: block; margin-top: 6px; }
    input, select { width: 100%; height: 42px; padding: 6px 10px; font-size: 16px; border: 1px solid #c7c7cc; border-radius: 8px; margin-top: 3px; background: #fff; color: #000; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
    .readonly-box { background: #e5e5ea !important; font-weight: bold; color: #000; }
    .btn { width: 100%; height: 46px; font-size: 16px; font-weight: bold; border: none; border-radius: 8px; cursor: pointer; margin-top: 10px; }
    .btn-add { background: #007aff; color: #fff; }
    .btn-export { background: #34c759; color: #fff; }
    .btn-clear { background: #ff3b30; color: #fff; }
    .msg-box { display: none; padding: 8px; border-radius: 6px; font-size: 13px; font-weight: bold; text-align: center; margin-top: 8px; }
    .msg-error { display: block; background: #fee2e2; color: #b91c1c; }
    .msg-success { display: block; background: #dcfce7; color: #15803d; }
    table { width: 100%; border-collapse: collapse; margin-top: 8px; font-size: 12px; }
    th, td { border: 1px solid #d1d1d6; padding: 6px 3px; text-align: center; }
    th { background: #f2f2f7; font-size: 11px; }
    .var-ok { color: #15803d; font-weight: bold; }
    .var-bad { color: #b91c1c; font-weight: bold; }

    @media print {
      body { background: #fff; padding: 0; }
      .card, .btn, label, select, input, h2, h3, .msg-box { display: none !important; }
      #printArea { display: block !important; }
      table { width: 100%; border-collapse: collapse; font-size: 10pt; }
      th, td { border: 1px solid #000 !important; padding: 4px; }
      th { background-color: #eee !important; }
    }
    #printArea { display: none; }
  </style>
</head>
<body>

  <div class="card">
    <h2>JYOTI GAS SERVICE, BORSAD.</h2>
    <h3>QC CHECK AT GODOWN</h3>

    <label>Cylinder Category</label>
    <select id="category" onchange="onCategoryChange()">
      <option value="14.2" selected>DOMESTIC 14.2KG</option>
      <option value="19.0">COMMERCIAL 19KG</option>
      <option value="5.0">FTL 5KG</option>
    </select>

    <div class="grid">
      <div>
        <label>Invoice No (Numeric Only)</label>
        <input type="text" inputmode="numeric" pattern="[0-9]*" id="invoiceNo" placeholder="e.g. 10452" oninput="saveHeader()">
      </div>
      <div>
        <label>Date</label>
        <input type="date" id="dateStr" oninput="saveHeader()">
      </div>
    </div>

    <div class="grid">
      <div>
        <label>Lorry No (Caps & Numbers)</label>
        <input type="text" autocapitalize="characters" autocorrect="off" autocomplete="off" spellcheck="false" id="lorryNo" placeholder="GJ-23-XX-1234" oninput="this.value = this.value.toUpperCase(); saveHeader();">
      </div>
      <div>
        <label>Date Received</label>
        <input type="date" id="dateReceived" oninput="saveHeader()">
      </div>
    </div>
  </div>

  <div class="card">
    <h3 style="text-align: left; margin: 0 0 6px 0; color: #111; border-bottom: 1px solid #eee; padding-bottom: 6px;">Add Cylinder Entry</h3>

    <div id="statusMsg" class="msg-box"></div>

    <label>Cylinder Serial No</label>
    <input type="text" inputmode="numeric" pattern="[0-9]*" id="cylSrNo" placeholder="e.g. 552565">

    <div class="grid">
      <div>
        <label>Tare Wt (KG)</label>
        <input type="text" inputmode="decimal" id="tareWt" placeholder="e.g. 16.0" oninput="runLiveCalc()">
      </div>
      <div>
        <label>Gross Wt (KG)</label>
        <input type="text" inputmode="decimal" id="grossWt" placeholder="e.g. 30.3" oninput="runLiveCalc()">
      </div>
    </div>

    <div class="grid">
      <div>
        <label>Fixed Net (KG)</label>
        <input type="text" id="netWt" class="readonly-box" value="14.2" readonly>
      </div>
      <div>
        <label>Live Variation (GMS)</label>
        <input type="text" id="variation" class="readonly-box" placeholder="0" readonly>
      </div>
    </div>

    <label>Remarks</label>
    <input type="text" id="remarks" placeholder="Optional (e.g. OK)">

    <button type="button" class="btn btn-add" id="btnAddCylinder" onclick="submitCylinder()">+ Add Cylinder</button>
  </div>

  <div class="card">
    <h3 style="text-align: left; margin: 0 0 6px 0; color: #111;">Saved Records (<span id="countSpan">0</span> / 36)</h3>
    <div style="overflow-x: auto;">
      <table>
        <thead>
          <tr>
            <th style="width: 26px;">#</th>
            <th>Cylinder No</th>
            <th>Tare</th>
            <th>Gross</th>
            <th>Net</th>
            <th>Var (GMS)</th>
            <th>Remarks</th>
          </tr>
        </thead>
        <tbody id="tableRows"></tbody>
      </table>
    </div>

    <button type="button" class="btn btn-export" onclick="triggerPrint()">📄 Print / Save PDF Report</button>
    <button type="button" class="btn btn-clear" onclick="resetAllData()">🗑️ Clear Batch</button>
  </div>

  <div id="printArea"></div>

  <script>
    var entries = [];

    function safeStore(key, val) {
      try { if (window.localStorage) localStorage.setItem(key, val); } catch (e) {}
    }
    function safeRead(key) {
      try { return window.localStorage ? localStorage.getItem(key) : null; } catch (e) { return null; }
    }

    function showStatus(text, isError) {
      var box = document.getElementById('statusMsg');
      box.className = isError ? 'msg-box msg-error' : 'msg-box msg-success';
      box.innerText = text;
      setTimeout(function() { box.className = 'msg-box'; }, 3000);
    }

    function onCategoryChange() {
      var catVal = parseFloat(document.getElementById('category').value) || 14.2;
      document.getElementById('netWt').value = catVal.toFixed(1);
      runLiveCalc();
      saveHeader();
    }

    function runLiveCalc() {
      var tStr = (document.getElementById('tareWt').value || '').replace(',', '.');
      var gStr = (document.getElementById('grossWt').value || '').replace(',', '.');
      var catVal = parseFloat(document.getElementById('category').value) || 14.2;
      var varElem = document.getElementById('variation');

      var t = parseFloat(tStr);
      var g = parseFloat(gStr);

      if (!isNaN(t) && !isNaN(g) && t > 0 && g > 0) {
        var diffGms = Math.round((g - t - catVal) * 1000);
        varElem.value = (diffGms > 0 ? "+" : "") + diffGms;
        varElem.style.color = Math.abs(diffGms) > 150 ? "#b91c1c" : "#15803d";
      } else {
        varElem.value = "";
        varElem.style.color = "#000";
      }
    }

    function submitCylinder() {
      var cyl = (document.getElementById('cylSrNo').value || '').trim();
      var tStr = (document.getElementById('tareWt').value || '').replace(',', '.');
      var gStr = (document.getElementById('grossWt').value || '').replace(',', '.');
      var catVal = parseFloat(document.getElementById('category').value) || 14.2;
      var rem = (document.getElementById('remarks').value || '').trim() || "OK";

      if (!cyl) {
        showStatus("Please enter Cylinder Serial Number!", true);
        return;
      }

      var t = parseFloat(tStr);
      var g = parseFloat(gStr);

      if (isNaN(t) || isNaN(g) || t <= 0 || g <= 0) {
        showStatus("Please enter valid Tare and Gross weights!", true);
        return;
      }

      var diffGms = Math.round((g - t - catVal) * 1000);
      var diffFormatted = (diffGms > 0 ? "+" : "") + diffGms;

      entries.push({
        sr: entries.length + 1,
        cyl: cyl,
        tare: t.toFixed(2),
        gross: g.toFixed(2),
        net: catVal.toFixed(1),
        varGms: diffFormatted,
        rem: rem
      });

      safeStore('jyoti_qc_records', JSON.stringify(entries));
      drawTable();

      document.getElementById('cylSrNo').value = "";
      document.getElementById('tareWt').value = "";
      document.getElementById('grossWt').value = "";
      document.getElementById('variation').value = "";
      document.getElementById('remarks').value = "";

      showStatus("Saved Cylinder #" + cyl + " successfully!", false);
    }

    function drawTable() {
      var tbody = document.getElementById('tableRows');
      tbody.innerHTML = "";
      for (var i = 0; i < entries.length; i++) {
        var item = entries[i];
        var isAlert = Math.abs(parseInt(item.varGms)) > 150;
        var tr = document.createElement('tr');
        tr.innerHTML =
          "<td>" + item.sr + "</td>" +
          "<td><b>" + item.cyl + "</b></td>" +
          "<td>" + item.tare + "</td>" +
          "<td>" + item.gross + "</td>" +
          "<td>" + item.net + "</td>" +
          "<td class='" + (isAlert ? "var-bad" : "var-ok") + "'>" + item.varGms + "</td>" +
          "<td>" + item.rem + "</td>";
        tbody.appendChild(tr);
      }
      document.getElementById('countSpan').innerText = entries.length;
    }

    function saveHeader() {
      safeStore('j_inv', document.getElementById('invoiceNo').value);
      safeStore('j_lorry', document.getElementById('lorryNo').value);
      safeStore('j_cat', document.getElementById('category').value);
      safeStore('j_date', document.getElementById('dateStr').value);
      safeStore('j_recv', document.getElementById('dateReceived').value);
    }

    function resetAllData() {
      if (confirm("Clear all rows and start a fresh batch?")) {
        entries = [];
        safeStore('jyoti_qc_records', JSON.stringify([]));
        drawTable();
        showStatus("Batch cleared.", false);
      }
    }

    function triggerPrint() {
      if (entries.length === 0) {
        showStatus("Please add cylinder records before printing!", true);
        return;
      }

      var cat = document.getElementById('category');
      var catText = cat.options[cat.selectedIndex].text;
      var inv = document.getElementById('invoiceNo').value || "-";
      var lorry = document.getElementById('lorryNo').value || "-";
      var d1 = document.getElementById('dateStr').value || "-";
      var d2 = document.getElementById('dateReceived').value || "-";

      var html = "<div style='text-align:center;'><h2>QC CHECK AT GODOWN</h2><h3>" + catText + "</h3></div>" +
        "<table style='margin-bottom:8px; border:none;'>" +
        "<tr><td style='border:none; text-align:left;'><b>INVOICE NO:</b> " + inv + "</td><td style='border:none; text-align:right;'><b>JYOTI GAS SERVICE, BORSAD.</b></td></tr>" +
        "<tr><td style='border:none; text-align:left;'><b>DATE:</b> " + d1 + "</td><td style='border:none; text-align:right;'><b>LORRY NO:</b> " + lorry + "</td></tr>" +
        "<tr><td style='border:none;'></td><td style='border:none; text-align:right;'><b>DATE OF RECEIVED:</b> " + d2 + "</td></tr>" +
        "</table>" +
        "<table><thead><tr>" +
        "<th>SR.NO</th><th>SR.NO OF CYLINDER</th><th>TARE WT. (KG)</th><th>GROSS WT. (KG)</th><th>NET WT. (KG)</th><th>VARIATION (GMS)</th><th>REMARKS</th>" +
        "</tr></thead><tbody>";

      for (var i = 0; i < 36; i++) {
        if (i < entries.length) {
          var e = entries[i];
          html += "<tr>" +
            "<td>" + e.sr + "</td>" +
            "<td>" + e.cyl + "</td>" +
            "<td>" + e.tare + "</td>" +
            "<td>" + e.gross + "</td>" +
            "<td>" + e.net + "</td>" +
            "<td>" + e.varGms + "</td>" +
            "<td>" + e.rem + "</td>" +
          "</tr>";
        } else {
          html += "<tr><td>" + (i + 1) + "</td><td></td><td></td><td></td><td></td><td></td><td></td></tr>";
        }
      }

      html += "</tbody></table>" +
        "<div style='display:flex; justify-content:space-between; margin-top:35px; font-weight:bold; font-size:12px;'>" +
        "<div>SIGN. OF GODOWN KEEPER</div><div>SIGN. OF DISTRIBUTOR</div></div>";

      document.getElementById('printArea').innerHTML = html;
      window.print();
    }

    (function init() {
      var today = new Date().toISOString().split('T')[0];
      var savedInv = safeRead('j_inv');
      var savedLorry = safeRead('j_lorry');
      var savedCat = safeRead('j_cat');
      var savedDate = safeRead('j_date');
      var savedRecv = safeRead('j_recv');

      if (savedInv) document.getElementById('invoiceNo').value = savedInv;
      if (savedLorry) document.getElementById('lorryNo').value = savedLorry;
      if (savedCat) document.getElementById('category').value = savedCat;
      document.getElementById('dateStr').value = savedDate || today;
      document.getElementById('dateReceived').value = savedRecv || today;

      var savedRecs = safeRead('jyoti_qc_records');
      if (savedRecs) {
        try { entries = JSON.parse(savedRecs) || []; } catch(e) { entries = []; }
      }

      onCategoryChange();
      drawTable();
    })();
  </script>
</body>
</html>
