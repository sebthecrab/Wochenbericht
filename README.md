<!-- 
============================================================
Copyright © 2025 Sebastian Holländer
Alle Rechte vorbehalten.

Kontakt: Sebastian.j.hollaender@gmail.com
============================================================
-->

<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Wochen Kurzbericht</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    .copy-success { display:none; }
    .required:after { content:" *"; color:red; }
    .badge-toggle { cursor: pointer; user-select: none; background-color: var(--bs-secondary-bg); color: var(--bs-body-color); border: 1px solid var(--bs-border-color); transition: all .15s ease-in-out; }
    .badge-toggle:hover { filter: brightness(0.97); }
    .btn-check:checked + .badge-toggle { background-color: var(--bs-primary); color: #fff; border-color: var(--bs-primary); }
    .customer-card { background:#fff; }
    .table-sm td, .table-sm th { padding: .25rem .5rem; }
    #copyAlert.sticky-alert { position: sticky; top: 0; z-index: 1056; }
    #printArea { display: none; }

    /* Druck-Optimierung */
    @page { margin: 10mm; }
    @media print {
      .report-card, .report-table, .print-block, .rollup-block,
      .report-table thead, .report-table tbody, .report-table tr, .report-table td, .report-table th {
        break-inside: avoid;
        page-break-inside: avoid;
      }
      h1, h2, h3, h4, h5, h6 {
        break-after: avoid-page;
        page-break-after: avoid;
      }
    }
  </style>
</head>
<body class="bg-light">

<div class="container py-5">
  <div class="row justify-content-center">
    <div class="col-12 col-xl-10">
      <h1 class="h4 mb-2">Wochen Kurzbericht</h1>

      <!-- Obere Buttons -->
      <div class="d-grid gap-2 gap-sm-2 d-sm-flex justify-content-sm-end mb-3">
        <button id="addCustomer" type="button" class="btn btn-success w-100 w-sm-auto">Kunden hinzufügen</button>
        <button id="saveAll" type="button" class="btn btn-outline-primary w-100 w-sm-auto">Alles speichern</button>
        <button id="makeAllReport" type="button" class="btn btn-primary w-100 w-sm-auto">Report erstellen</button>
      </div>

      <div id="customersContainer" class="d-flex flex-column gap-4"></div>

      <!-- Untere Buttons -->
      <div class="d-grid gap-2 gap-sm-2 d-sm-flex justify-content-sm-end my-3">
        <button id="addCustomerBottom" type="button" class="btn btn-success w-100 w-sm-auto">Kunden hinzufügen</button>
        <button id="saveAllBottom" type="button" class="btn btn-outline-primary w-100 w-sm-auto">Alles speichern</button>
        <button id="makeAllReportBottom" type="button" class="btn btn-primary w-100 w-sm-auto">Report erstellen</button>
      </div>

      <!-- Cookie-Optionen -->
      <div class="mt-3">
        <div class="alert alert-info py-2 mb-3" role="alert">
          <strong>Hinweis:</strong> Für wiederholte Nutzung wird empfohlen, die passenden Cookie-Optionen zu aktivieren.
        </div>
        <div class="form-check mb-2">
          <input class="form-check-input" type="checkbox" id="storeValues">
          <label class="form-check-label" for="storeValues">Prüf- und Fehlerwerte in Cookies speichern</label>
        </div>
        <div class="form-check mb-2">
          <input class="form-check-input" type="checkbox" id="storeNames">
          <label class="form-check-label" for="storeNames">Personennamen in Cookies speichern</label>
        </div>
        <div class="form-check mb-2">
          <input class="form-check-input" type="checkbox" id="storeCustomer">
          <label class="form-check-label" for="storeCustomer">Kundendaten in Cookies speichern</label>
        </div>
      </div>

    </div>
  </div>
</div>

<!-- PRINT AREA (für PDF) -->
<div id="printArea"></div>

<!-- REPORT MODAL -->
<div class="modal fade" id="reportModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-fullscreen">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Report</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        <!-- Sticky Copy-Alert -->
        <div id="copyAlert" class="alert alert-success d-none py-2 px-3 sticky-alert" role="alert">
          Daten wurden in die Zwischenablage übernommen.
        </div>

        <div class="d-flex justify-content-between align-items-center flex-wrap gap-2">
          <div id="globalKWDisplay" class="fw-semibold"></div>
          <div class="d-flex gap-2">
            <button id="copyAllBtn" type="button" class="btn btn-sm btn-outline-dark">Report kopieren</button>
            <button id="printBtn" type="button" class="btn btn-sm btn-dark">Als PDF drucken</button>
          </div>
        </div>
        <small id="copyAllSuccess" class="text-success copy-success">Report in die Zwischenablage kopiert.</small>
        <hr>

        <div id="combinedReportContent"></div>

        <hr id="rollupHr" />
        <div id="personsRollupSection" class="rollup-block">
          <h6 class="mb-2" id="rollupHeading">Gesamtüberblick Personen (kundenübergreifend)</h6>
          <div class="table-responsive">
            <table class="table table-sm table-bordered mb-2 report-table avoid-break">
              <thead>
                <tr>
                  <th>Person</th>
                  <th>Prüfungen</th>
                  <th>Nicht bestanden</th>
                  <th>Fehlerquote</th>
                </tr>
              </thead>
              <tbody id="personsRollupTable"></tbody>
            </table>
          </div>
        </div>

        <hr>
        <p id="pGesamt"><strong>GESAMT Prüfungen (alle Kunden):</strong> <span id="allGesamtSumme"></span></p>
        <p id="pFehler"><strong>GESAMT Nicht bestanden (alle Kunden):</strong> <span id="allFehlerSumme"></span></p>
        <p id="pQuote"><strong>GESAMT Fehlerquote (alle Kunden):</strong> <span id="allFehlerQuoteGesamt"></span></p>

        <!-- Buttons am Ende -->
        <div class="d-flex justify-content-end gap-2 mt-3 flex-wrap">
          <button id="copyAllBtnBottom" type="button" class="btn btn-sm btn-outline-dark">Report kopieren</button>
          <button id="printBtnBottom" type="button" class="btn btn-sm btn-dark">Als PDF drucken</button>
        </div>

        <pre id="combinedReportText" class="visually-hidden"></pre>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Schließen</button>
      </div>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
<script>
/* ===== Storage-Helpers (localStorage anstelle Cookies) ===== */
function setCookie(key, valueJson, days) {
  try { localStorage.setItem(key, valueJson); } catch(e) {}
}
function getCookie(key) {
  try { return localStorage.getItem(key); } catch(e) { return null; }
}
function clearAllCookies() {
  try {
    Object.keys(localStorage).forEach(function(k){
      if (k.startsWith('wk_')) localStorage.removeItem(k);
    });
  } catch(e) {}
}

/* ===== Utils ===== */
function escapeHTML(str){
  if (str === null || str === undefined) return '';
  return String(str).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/"/g, '&quot;').replace(/'/g, '&#039;');
}
function getMondayOfCurrentWeek(date) {
  if (!date) date = new Date();
  var d = new Date(date);
  var day = d.getDay();
  var diff = (day === 0 ? -6 : 1 - day);
  d.setDate(d.getDate() + diff);
  d.setHours(0,0,0,0);
  return d;
}
function getISOWeek(d) {
  if (!d) d = new Date();
  var date = new Date(Date.UTC(d.getFullYear(), d.getMonth(), d.getDate()));
  var dayNum = date.getUTCDay() || 7;
  date.setUTCDate(date.getUTCDate() + 4 - dayNum);
  var yearStart = new Date(Date.UTC(date.getUTCFullYear(),0,1));
  var weekNo = Math.ceil((((date - yearStart) / 86400000) + 1) / 7);
  return { week: weekNo, year: date.getUTCFullYear() };
}
function formatDE(date) { return date.toLocaleDateString('de-DE'); }
function weekdayDE(date) { return date.toLocaleDateString('de-DE', { weekday: 'long' }); }

// Zahlen nur ganzzahlig
function normalizeIntegerInput(el) {
  if (!el || el.value === '') return;
  var num = Number(String(el.value).replace(',', '.'));
  if (!isFinite(num)) { el.value=''; return; }
  num = Math.round(num);
  var min = el.hasAttribute('min') ? Number(el.getAttribute('min')) : undefined;
  var max = el.hasAttribute('max') ? Number(el.getAttribute('max')) : undefined;
  if (isFinite(min) && num < min) num = min;
  if (isFinite(max) && num > max) num = max;
  el.value = String(num);
}
function attachIntegerHandlers(scope) {
  if (!scope) scope = document;
  var ints = scope.querySelectorAll('.integer-only');
  ints.forEach(function(el){
    el.addEventListener('input', function(){ el.value = el.value.replace(/[^0-9]/g, ''); });
    el.addEventListener('blur', function(){ normalizeIntegerInput(el); });
    el.addEventListener('change', function(){ normalizeIntegerInput(el); });
  });
}

/* ===== Kunden-UI ===== */
function createCustomerCard(index, customerData) {
  customerData = customerData || {};
  var nameVal = customerData.name ? String(customerData.name) : '';
  var numberVal = customerData.number ? String(customerData.number) : '';
  var orderVal = customerData.order ? String(customerData.order) : '';
  var personsArr = (customerData.persons && Array.isArray(customerData.persons)) ? customerData.persons : [''];
  var selectedDays = (customerData.selectedDays && Array.isArray(customerData.selectedDays) && customerData.selectedDays.length===5) ? customerData.selectedDays : [true,true,true,true,false];

  var card = document.createElement('div');
  card.className = 'card customer-card shadow-sm';
  card.dataset.customerIndex = index;
  card.innerHTML = ''
    + '<div class="card-header d-flex justify-content-between align-items-center">'
    + '  <div><strong>Kunde</strong> <span class="cust-number">#' + index + '</span></div>'
    + '  <div class="d-flex gap-2">'
    + '    <button type="button" class="btn btn-sm btn-outline-secondary" data-action="copy-interim">Zwischenreport kopieren</button>'  /* === Zwischenreport Button === */
    + '    <button type="button" class="btn btn-sm btn-outline-danger" data-action="remove-customer">Kunden entfernen</button>'
    + '  </div>'
    + '</div>'
    + '<div class="card-body">'
    + '  <div class="row g-3 mb-3">'
    + '    <div class="col-12 col-md-6">'
    + '      <label class="form-label">Kundenname</label>'
    + '      <input type="text" class="form-control kundenName" placeholder="Optional" value="' + nameVal.replace(/"/g,'&quot;') + '">'
    + '    </div>'
    + '    <div class="col-12 col-md-3">'
    + '      <label class="form-label">Kundennummer</label>'
    + '      <input type="text" class="form-control kundenNummer" placeholder="Optional" value="' + numberVal.replace(/"/g,'&quot;') + '">'
    + '    </div>'
    + '    <div class="col-12 col-md-3">'
    + '      <label class="form-label">Auftragsnummer</label>'
    + '      <input type="text" class="form-control auftragNummer" placeholder="Optional" value="' + orderVal.replace(/"/g,'&quot;') + '">'
    + '    </div>'
    + '    <div class="col-12">'
    + '      <label class="form-label mb-2">Anwesenheitstage beim Kunden (aktuelle Kalenderwoche)</label>'
    + '      <div class="dayCheckboxes d-flex flex-wrap gap-2"></div>'
    + '    </div>'
    + '  </div>'
    + '  <div class="personenContainer"></div>'
    + '  <div class="d-grid gap-2 mb-2">'
    + '    <button type="button" class="btn btn-secondary" data-action="addPerson">Weitere Person hinzufügen</button>'
    + '  </div>'
    + '  <small class="text-success d-none interim-copy-success">Zwischenreport in die Zwischenablage kopiert.</small>' /* === Zwischenreport Erfolgshinweis === */
    + '</div>';

  initWeekUIForCard(card, selectedDays);
  for (var i=0; i<personsArr.length; i++) {
    addPersonToCard(card, i+1, personsArr[i] || '', customerData.storeValues ? customerData.values && customerData.values[i] : null);
  }
  attachIntegerHandlers(card);
  return card;
}
function initWeekUIForCard(card, selectedDays) {
  if (!selectedDays) selectedDays = [true,true,true,true,false];
  var monday = getMondayOfCurrentWeek(new Date());
  var dayContainer = card.querySelector('.dayCheckboxes');
  dayContainer.innerHTML = '';

  for (var i=0; i<5; i++) {
    var d = new Date(monday);
    d.setDate(monday.getDate() + i);
    var id = 'wkday_' + Date.now() + '_' + Math.random().toString(36).slice(2);
    var labelText = weekdayDE(d);
    labelText = labelText.charAt(0).toUpperCase() + labelText.slice(1) + ' (' + formatDE(d) + ')';

    var input = document.createElement('input');
    input.type = 'checkbox';
    input.className = 'btn-check wkday';
    input.id = id;
    input.autocomplete = 'off';
    input.setAttribute('data-date', d.toISOString());
    if (selectedDays[i]) input.checked = true;

    var label = document.createElement('label');
    label.className = 'badge rounded-pill badge-toggle px-3 py-2';
    label.setAttribute('for', id);
    label.textContent = labelText;

    dayContainer.appendChild(input);
    dayContainer.appendChild(label);
  }
  dayContainer.addEventListener('change', function(){ saveAllToCookies(); });
}
function addPersonToCard(card, index, nameValue, valueObj) {
  var container = card.querySelector('.personenContainer');
  var wrapper = document.createElement('div');
  wrapper.className = 'person mb-3 p-3 border rounded';
  var gesVal = valueObj && typeof valueObj.gesamt === 'number' ? String(valueObj.gesamt) : '';
  var fehVal = valueObj && typeof valueObj.fehler === 'number' ? String(valueObj.fehler) : '';
  wrapper.innerHTML = ''
    + '<div class="d-flex justify-content-between align-items-center mb-2">'
    + '  <h5 class="mb-0">Person ' + index + '</h5>'
    + '  <div class="d-flex gap-2">'
    + '    <button type="button" class="btn btn-sm btn-outline-danger" data-action="removePerson">Entfernen</button>'
    + '  </div>'
    + '</div>'
    + '<div class="mb-3">'
    + '  <label class="form-label">Name (optional)</label>'
    + '  <input type="text" class="form-control name" placeholder="Name der Person" value="' + (nameValue ? String(nameValue).replace(/"/g,'&quot;') : '') + '">'
    + '</div>'
    + '<div class="mb-3">'
    + '  <label class="form-label required">Anzahl der Prüfungen</label>'
    + '  <input type="number" class="form-control gesamt integer-only" placeholder="z. B. 50" min="0" step="1" inputmode="numeric" pattern="\\d*" required value="' + gesVal + '">'
    + '</div>'
    + '<div class="mb-3">'
    + '  <label class="form-label required">Anzahl der nicht bestandenen Prüfungen</label>'
    + '  <input type="number" class="form-control fehler integer-only" placeholder="z. B. 7" min="0" step="1" inputmode="numeric" pattern="\\d*" required value="' + fehVal + '">'
    + '</div>';
  container.appendChild(wrapper);
  attachIntegerHandlers(wrapper);
}
function renumberPersons(card) {
  var persons = card.querySelectorAll('.personenContainer .person');
  persons.forEach(function(p, idx){
    var h5 = p.querySelector('h5');
    if (h5) h5.textContent = 'Person ' + (idx + 1);
  });
}

/* ===== Daten sammeln/speichern ===== */
function collectAllData() {
  var storeValues = document.getElementById('storeValues').checked;
  var storeNames = document.getElementById('storeNames').checked;
  var storeCustomer = document.getElementById('storeCustomer').checked;

  var customers = [];
  document.querySelectorAll('#customersContainer .customer-card').forEach(function(card){
    var name = (card.querySelector('.kundenName') || {value:''}).value.trim();
    var number = (card.querySelector('.kundenNummer') || {value:''}).value.trim();
    var order = (card.querySelector('.auftragNummer') || {value:''}).value.trim();

    var persons = [];
    var values = [];
    var personBlocks = card.querySelectorAll('.personenContainer .person');
    personBlocks.forEach(function(block){
      var nm = (block.querySelector('.name')||{value:''}).value.trim();
      var g = parseInt((block.querySelector('.gesamt')||{value:''}).value, 10);
      var f = parseInt((block.querySelector('.fehler')||{value:''}).value, 10);

      if (storeNames) { persons.push(nm); }
      if (storeValues && storeNames) {
        values.push({ gesamt: isNaN(g)? null : g, fehler: isNaN(f)? null : f });
      }
    });

    var selected = [];
    card.querySelectorAll('.dayCheckboxes .btn-check.wkday').forEach(function(cb){ selected.push(!!cb.checked); });

    customers.push({
      name: storeCustomer ? name : '',
      number: storeCustomer ? number : '',
      order: storeCustomer ? order : '',
      persons: persons,
      selectedDays: selected,
      storeValues: storeValues && storeNames,
      values: values
    });
  });

  if (!document.getElementById('storeCustomer').checked) { customers = []; }

  return { storeValues: storeValues, storeNames: storeNames, storeCustomer: storeCustomer, customers: customers };
}
function saveAllToCookies() {
  var data = collectAllData();
  if (!data.storeValues && !data.storeNames && !data.storeCustomer) {
    clearAllCookies();
    return;
  }
  setCookie('wk_customers_v3', JSON.stringify(data));
}
function loadAllFromCookies() {
  var raw = getCookie('wk_customers_v3');
  if (!raw) return null;
  try { return JSON.parse(raw); } catch(e) { return null; }
}

/* ===== Init & UI-Aufbau ===== */
var customersContainer = document.getElementById('customersContainer');

function buildUIFromData(payload) {
  customersContainer.innerHTML = '';

  var storeValues = false, storeNames = false, storeCustomer = false, customersData = null;
  if (payload && typeof payload === 'object') {
    if (typeof payload.storeValues === 'boolean') storeValues = payload.storeValues;
    if (typeof payload.storeNames === 'boolean') storeNames = payload.storeNames;
    if (typeof payload.storeCustomer === 'boolean') storeCustomer = payload.storeCustomer;
    customersData = Array.isArray(payload.customers) ? payload.customers : null;
  }

  document.getElementById('storeValues').checked = storeValues;
  document.getElementById('storeNames').checked = storeNames;
  document.getElementById('storeCustomer').checked = storeCustomer;

  if (!customersData || customersData.length === 0) {
    customersContainer.appendChild(createCustomerCard(1, { name:'', number:'', order:'', persons:[''], selectedDays:[true,true,true,true,false], storeValues:false, values:[] }));
  } else {
    customersData.forEach(function(cust, idx){
      cust.storeValues = storeValues && storeNames;
      customersContainer.appendChild(createCustomerCard(idx+1, cust));
    });
  }
}

function duplicatePersonsFromFirstCustomer() {
  var firstCard = customersContainer.querySelector('.customer-card');
  if (!firstCard) return [''];
  var names = [];
  firstCard.querySelectorAll('.personenContainer .person .name').forEach(function(inp){
    names.push((inp.value || '').trim());
  });
  return names.length ? names : [''];
}

/* ===== Aktionen ===== */
function addCustomerAction(){
  var newIndex = customersContainer.querySelectorAll('.customer-card').length + 1;
  var personsTemplate = duplicatePersonsFromFirstCustomer();
  var firstCard = customersContainer.querySelector('.customer-card');
  var selectedDays = [true,true,true,true,false];
  if (firstCard) {
    selectedDays = Array.from(firstCard.querySelectorAll('.dayCheckboxes .btn-check.wkday')).map(function(cb){ return !!cb.checked; });
  }
  var card = createCustomerCard(newIndex, {
    name:'', number:'', order:'', persons: personsTemplate, selectedDays: selectedDays,
    storeValues: (document.getElementById('storeValues').checked && document.getElementById('storeNames').checked),
    values: []
  });
  customersContainer.appendChild(card);
  saveAllToCookies();
}
function saveAllAction(btn){
  saveAllToCookies();
  if (btn) { var t = btn.textContent; btn.textContent = 'Gespeichert ✓'; setTimeout(function(){ btn.textContent = t; }, 1200); }
}
function openReportAction(){ runReport(); }

document.getElementById('addCustomer').addEventListener('click', addCustomerAction);
document.getElementById('saveAll').addEventListener('click', function(){ saveAllAction(document.getElementById('saveAll')); });
document.getElementById('makeAllReport').addEventListener('click', openReportAction);
document.getElementById('addCustomerBottom').addEventListener('click', addCustomerAction);
document.getElementById('saveAllBottom').addEventListener('click', function(){ saveAllAction(document.getElementById('saveAllBottom')); });
document.getElementById('makeAllReportBottom').addEventListener('click', openReportAction);

/* Delegation in Karten */
customersContainer.addEventListener('click', function(e){
  var action = null, target = e.target;
  while (target && target !== customersContainer) {
    if (target.dataset && target.dataset.action) { action = target.dataset.action; break; }
    target = target.parentElement;
  }
  if (!action) return;
  var card = (target && target.closest) ? target.closest('.customer-card') : null;
  if (!card) return;

  if (action === 'remove-customer') {
    card.remove();
    customersContainer.querySelectorAll('.customer-card').forEach(function(c, idx){
      c.dataset.customerIndex = idx+1;
      var no = c.querySelector('.cust-number'); if (no) no.textContent = '#' + (idx+1);
    });
    saveAllToCookies();
  }
  if (action === 'addPerson') {
    var count = card.querySelectorAll('.personenContainer .person').length;
    addPersonToCard(card, count+1, '');
    saveAllToCookies();
  }
  if (action === 'removePerson') {
    var p = target.closest('.person'); if (p) p.remove();
    renumberPersons(card);
    saveAllToCookies();
  }

  /* === Zwischenreport: Button-Handler === */
  if (action === 'copy-interim') {
    copyInterimReport(card);
  }
});
customersContainer.addEventListener('input', function(e){
  if (e.target && e.target.closest('.customer-card')) { saveAllToCookies(); }
});
document.getElementById('storeValues').addEventListener('change', saveAllToCookies);
document.getElementById('storeNames').addEventListener('change', saveAllToCookies);
document.getElementById('storeCustomer').addEventListener('change', saveAllToCookies);

/* ===== Report ===== */
function runReport(){
  var modalEl = document.getElementById('reportModal');
  var modal = new bootstrap.Modal(modalEl);

  document.getElementById('combinedReportContent').innerHTML = '';
  document.getElementById('personsRollupTable').innerHTML = '';
  document.getElementById('combinedReportText').textContent = '';
  document.getElementById('allGesamtSumme').textContent = '0';
  document.getElementById('allFehlerSumme').textContent = '0';
  document.getElementById('allFehlerQuoteGesamt').textContent = '0.00 %';
  document.getElementById('personsRollupSection').style.display = '';
  document.getElementById('rollupHr').style.display = '';
  var alertEl = document.getElementById('copyAlert'); if (alertEl) { alertEl.classList.add('d-none'); }

  var iso = getISOWeek(new Date());
  var kwStr = 'Kalenderwoche: KW ' + iso.week + '/' + iso.year;
  var kwEl = document.getElementById('globalKWDisplay'); if (kwEl) kwEl.textContent = kwStr;
  document.title = 'Wochen Kurzbericht – KW ' + iso.week + '/' + iso.year;

  var allGesamt = 0, allFehler = 0;
  var combinedHTML = '', combinedText = '', personsRollup = {};

  var cards = document.querySelectorAll('#customersContainer .customer-card');
  if (!cards.length) { alert('Bitte mindestens einen Kunden anlegen.'); return; }

  for (var c=0; c<cards.length; c++) {
    var card = cards[c];
    var kundenname = escapeHTML(((card.querySelector('.kundenName')||{value:''}).value || '').trim() || '-');
    var kundennr = escapeHTML(((card.querySelector('.kundenNummer')||{value:''}).value || '').trim() || '-');
    var auftrag = escapeHTML(((card.querySelector('.auftragNummer')||{value:''}).value || '').trim() || '-');

    var checkedLabels = [];
    card.querySelectorAll('.btn-check.wkday:checked').forEach(function(cb){
      var lbl = cb.nextElementSibling ? cb.nextElementSibling.textContent : '';
      if (lbl) checkedLabels.push(lbl);
    });
    var checkedDaysText = escapeHTML(checkedLabels.join(', '));

    var gesamtSum = 0, fehlerSum = 0;
    var personsRows = '', personsText = '';

    var personBlocks = card.querySelectorAll('.personenContainer .person');
    if (!personBlocks.length) { alert('Bitte beim Kunden #' + (c+1) + ' mindestens eine Person hinzufügen.'); return; }

    for (var i=0; i<personBlocks.length; i++) {
      var block = personBlocks[i];
      var nameRaw = (block.querySelector('.name')||{value:''}).value.trim() || ('Person ' + (i+1));
      var name = escapeHTML(nameRaw);
      var gesamtInput = block.querySelector('.gesamt');
      var fehlerInput = block.querySelector('.fehler');

      normalizeIntegerInput(gesamtInput);
      normalizeIntegerInput(fehlerInput);

      var gesamt = parseInt((gesamtInput||{value:''}).value, 10);
      var fehler = parseInt((fehlerInput||{value:''}).value, 10);

      if (!isFinite(gesamt) || !isFinite(fehler)) { alert('Bitte beim Kunden #' + (c+1) + ' alle Pflichtfelder ausfüllen.'); return; }
      if (fehler > gesamt) { alert('Wert bei ' + nameRaw + ' (Kunde #' + (c+1) + '): "Nicht bestanden" darf nicht größer als "Prüfungen" sein!'); return; }

      var q = gesamt > 0 ? (fehler / gesamt) * 100 : 0;
      personsRows += '<tr><td>'+name+'</td><td>'+gesamt+'</td><td>'+fehler+'</td><td>'+q.toFixed(2)+' %</td></tr>';
      personsText += nameRaw + ': Prüfungen: ' + gesamt + ', Nicht bestanden: ' + fehler + ', Fehlerquote: ' + q.toFixed(2) + ' %\n';

      gesamtSum += gesamt; fehlerSum += fehler;
      if (!personsRollup[nameRaw]) personsRollup[nameRaw] = { pruef: 0, fehler: 0 };
      personsRollup[nameRaw].pruef += gesamt;
      personsRollup[nameRaw].fehler += fehler;
    }

    var qGes = gesamtSum > 0 ? (fehlerSum / gesamtSum) * 100 : 0;
    var collapseId = 'custCollapse_' + (c+1);
    combinedHTML += ''
      + '<div class="card mb-3 report-card avoid-break">'
      + '  <div class="print-block">'
      + '    <div class="card-header">'
      + '      <a class="text-decoration-none d-block" data-bs-toggle="collapse" href="#'+collapseId+'" role="button" aria-expanded="true" aria-controls="'+collapseId+'">'
      + '        <strong>Kunde #'+(c+1) + (kundenname !== '-' ? (': ' + kundenname) : '') + '</strong>'
      + '      </a>'
      + '    </div>'
      + '    <div id="'+collapseId+'" class="collapse show">'
      + '      <div class="card-body">'
      + '        <div class="row mb-2">'
      + '          <div class="col-12 col-md-6"><strong>Kundennummer:</strong> ' + kundennr + '</div>'
      + '          <div class="col-12 col-md-6"><strong>Auftragsnummer:</strong> ' + auftrag + '</div>'
      + '          <div class="col-12"><strong>Anwesenheitstage:</strong> ' + (checkedDaysText || '-') + '</div>'
      + '        </div>'
      + '        <div class="table-responsive">'
      + '          <table class="table table-sm table-bordered mb-2 report-table avoid-break">'
      + '            <thead><tr><th>Person</th><th>Prüfungen</th><th>Nicht bestanden</th><th>Fehlerquote</th></tr></thead>'
      + '            <tbody>'+personsRows+'</tbody>'
      + '            <tfoot><tr><th>Summe</th><th>'+gesamtSum+'</th><th>'+fehlerSum+'</th><th>'+qGes.toFixed(2)+' %</th></tr></tfoot>'
      + '          </table>'
      + '        </div>'
      + '      </div>'
      + '    </div>'
      + '  </div>'
      + '</div>';

    combinedText += 'Kunde #' + (c+1) + (kundenname !== '-' ? (': ' + kundenname) : '') + '\n'
      + 'Kundennummer: ' + kundennr + '\n'
      + 'Auftragsnummer: ' + auftrag + '\n'
      + 'Anwesenheitstage: ' + (checkedDaysText || '-') + '\n'
      + personsText
      + 'Summe: Prüfungen: ' + gesamtSum + ', Nicht bestanden: ' + fehlerSum + ', Fehlerquote: ' + qGes.toFixed(2) + ' %\n\n';

    allGesamt += gesamtSum; allFehler += fehlerSum;
  }

  document.getElementById('combinedReportContent').innerHTML = combinedHTML;

  var isSingle = (cards.length === 1);
  var rollupSection = document.getElementById('personsRollupSection');
  var rollupHr = document.getElementById('rollupHr');

  var personsRollupText = '';
  var personsRollupHTML = '';
  if (isSingle) {
    rollupSection.style.display = 'none';
    rollupHr.style.display = 'none';
  } else {
    var rollupTableBody = document.getElementById('personsRollupTable');
    rollupTableBody.innerHTML = '';
    var tempRows = '';
    Object.keys(personsRollup).sort(function(a,b){ return a.localeCompare(b, 'de'); }).forEach(function(nameKey){
      var sums = personsRollup[nameKey];
      var q = sums.pruef > 0 ? (sums.fehler / sums.pruef) * 100 : 0;
      var rowHTML = '<tr><td>'+escapeHTML(nameKey)+'</td><td>'+sums.pruef+'</td><td>'+sums.fehler+'</td><td>'+q.toFixed(2)+' %</td></tr>';
      rollupTableBody.insertAdjacentHTML('beforeend', rowHTML);
      tempRows += rowHTML;
      personsRollupText += nameKey + ': Prüfungen: ' + sums.pruef + ', Nicht bestanden: ' + sums.fehler + ', Fehlerquote: ' + q.toFixed(2) + ' %\n';
    });
    personsRollupHTML = ''
      + '<div class="rollup-block">'
      + '  <h6 class="mb-2" style="break-after: avoid-page; page-break-after: avoid;">Gesamtüberblick Personen (kundenübergreifend)</h6>'
      + '  <div class="table-responsive">'
      + '    <table class="table table-sm table-bordered mb-2 report-table avoid-break">'
      + '      <thead><tr><th>Person</th><th>Prüfungen</th><th>Nicht bestanden</th><th>Fehlerquote</th></tr></thead>'
      + '      <tbody>'+tempRows+'</tbody>'
      + '    </table>'
      + '  </div>'
      + '</div>';
  }

  var allQuote = allGesamt > 0 ? (allFehler / allGesamt) * 100 : 0;
  document.getElementById('allGesamtSumme').textContent = allGesamt;
  document.getElementById('allFehlerSumme').textContent = allFehler;
  document.getElementById('allFehlerQuoteGesamt').textContent = allQuote.toFixed(2) + ' %';

  var pGesamt = document.getElementById('pGesamt');
  var pFehler = document.getElementById('pFehler');
  var pQuote  = document.getElementById('pQuote');
  if (isSingle) { pGesamt.style.display = 'none'; pFehler.style.display = 'none'; pQuote.style.display  = 'none'; }
  else { pGesamt.style.display = ''; pFehler.style.display = ''; pQuote.style.display  = ''; }

  var header = 'REPORT\n' + kwStr + '\n\n';
  var combinedCopy = header + combinedText;
  if (!isSingle && personsRollupText) combinedCopy += 'Gesamtüberblick Personen (kundenübergreifend)\n' + personsRollupText + '\n';
  if (!isSingle) combinedCopy += 'GESAMT (alle Kunden): Prüfungen: ' + allGesamt + ', Nicht bestanden: ' + allFehler + ', Fehlerquote: ' + allQuote.toFixed(2) + ' %';
  document.getElementById('combinedReportText').textContent = combinedCopy;

  // PrintArea füllen
  var printHTML = ''
    + '<div>'
    + '  <h2 class="h4">Report</h2>'
    + '  <div class="mb-2"><strong>'+ kwStr +'</strong></div>'
    +    document.getElementById('combinedReportContent').innerHTML
    +    (!isSingle ? ('<hr>' + personsRollupHTML) : '')
    + '  <hr>'
    + '  <div>'
    + '    <p><strong>GESAMT Prüfungen (alle Kunden):</strong> ' + (isSingle ? '' : allGesamt) + '</p>'
    + '    <p><strong>GESAMT Nicht bestanden (alle Kunden):</strong> ' + (isSingle ? '' : allFehler) + '</p>'
    + '    <p><strong>GESAMT Fehlerquote (alle Kunden):</strong> ' + (isSingle ? '' : (allQuote.toFixed(2) + ' %')) + '</p>'
    + '  </div>'
    + '</div>';
  document.getElementById('printArea').innerHTML = printHTML;

  var modal = new bootstrap.Modal(document.getElementById('reportModal'));
  modal.show();
}

// Kopieren (Gesamtreport)
function copyReport(){
  var text = (document.getElementById('combinedReportText')||{textContent:''}).textContent || '';
  if (!text.trim()) { alert('Bitte zuerst den Report erstellen.'); return; }
  function showAlert(){
    var info = document.getElementById('copyAllSuccess'); if (info) { info.style.display = 'inline'; setTimeout(function(){ info.style.display = 'none'; }, 2000); }
    var alertEl = document.getElementById('copyAlert'); if (alertEl) { alertEl.classList.remove('d-none'); setTimeout(function(){ alertEl.classList.add('d-none'); }, 3000); }
  }
  try {
    if (navigator.clipboard && navigator.clipboard.writeText) {
      navigator.clipboard.writeText(text).then(function(){ showAlert(); }, function(){ fallbackCopyAll(text, showAlert); });
    } else { fallbackCopyAll(text, showAlert); }
  } catch(e) { fallbackCopyAll(text, showAlert); }
}
function fallbackCopyAll(text, cb) {
  var ta = document.createElement('textarea');
  ta.value = text; document.body.appendChild(ta); ta.select();
  try { document.execCommand('copy'); } catch(e) {}
  document.body.removeChild(ta);
  if (typeof cb === 'function') cb();
}
document.getElementById('copyAllBtn').addEventListener('click', copyReport);
document.getElementById('copyAllBtnBottom').addEventListener('click', copyReport);

// Drucken in neuem Fenster (verhindert Leerseiten)
function printReport(){
  var pa = document.getElementById('printArea');
  if (!pa || !pa.innerHTML.trim()) { alert('Bitte zuerst den Report erstellen.'); return; }
  var w = window.open('', '_blank');
  var title = document.title || 'Report';
  var printCss = '@page{ margin:10mm; }'
    + 'body{ padding:6mm; }'
    + '.table-sm td,.table-sm th{ padding:.18rem .36rem; }'
    + '.report-card,.report-table,.report-table thead,.report-table tbody,.report-table tr,.report-table td,.report-table th,.print-block,.rollup-block{'
    + '  break-inside: avoid; page-break-inside: avoid;'
    + '}'
    + 'h1,h2,h3,h4,h5,h6{ break-after: avoid-page; page-break-after: avoid; }';
  w.document.open();
  w.document.write('<!DOCTYPE html><html lang="de"><head><meta charset="utf-8"><title>'+title+'</title>'
    + '<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css">'
    + '<style>'+printCss+'</style></head><body>'
    + pa.innerHTML + '</body></html>');
  w.document.close();
  w.focus();
  setTimeout(function(){ w.print(); setTimeout(function(){ w.close(); }, 300); }, 150);
}
document.getElementById('printBtn').addEventListener('click', printReport);
document.getElementById('printBtnBottom').addEventListener('click', printReport);

/* === Zwischenreport: Builder & Kopieren pro Kunde === */
function buildInterimReportForCard(card) {
  // KW bestimmen
  var iso = getISOWeek(new Date());
  var kwStr = 'Kalenderwoche: KW ' + iso.week + '/' + iso.year;

  // Kundendaten
  var kundennameRaw = ((card.querySelector('.kundenName')||{value:''}).value || '').trim() || '-';
  var kundenname = kundennameRaw;
  var kundennr = ((card.querySelector('.kundenNummer')||{value:''}).value || '').trim() || '-';
  var auftrag = ((card.querySelector('.auftragNummer')||{value:''}).value || '').trim() || '-';

  // Anwesenheitstage
  var checkedLabels = [];
  card.querySelectorAll('.btn-check.wkday:checked').forEach(function(cb){
    var lbl = cb.nextElementSibling ? cb.nextElementSibling.textContent : '';
    if (lbl) checkedLabels.push(lbl);
  });
  var checkedDaysText = checkedLabels.join(', ') || '-';

  // Personen & Summen
  var personBlocks = card.querySelectorAll('.personenContainer .person');
  if (!personBlocks.length) {
    throw new Error('Bitte mindestens eine Person beim Kunden erfassen.');
  }

  var gesamtSum = 0, fehlerSum = 0;
  var personsText = '';

  for (var i=0; i<personBlocks.length; i++) {
    var block = personBlocks[i];
    var nameRaw = (block.querySelector('.name')||{value:''}).value.trim() || ('Person ' + (i+1));
    var gesamtInput = block.querySelector('.gesamt');
    var fehlerInput = block.querySelector('.fehler');

    normalizeIntegerInput(gesamtInput);
    normalizeIntegerInput(fehlerInput);

    var gesamt = parseInt((gesamtInput||{value:''}).value, 10);
    var fehler = parseInt((fehlerInput||{value:''}).value, 10);

    if (!isFinite(gesamt) || !isFinite(fehler)) {
      throw new Error('Bitte alle Pflichtfelder (Prüfungen/Nicht bestanden) ausfüllen.');
    }
    if (fehler > gesamt) {
      throw new Error('Bei "' + nameRaw + '": "Nicht bestanden" darf nicht größer als "Prüfungen" sein.');
    }

    var q = gesamt > 0 ? (fehler / gesamt) * 100 : 0;
    personsText += nameRaw + ': Prüfungen: ' + gesamt + ', Nicht bestanden: ' + fehler + ', Fehlerquote: ' + q.toFixed(2) + ' %\n';
    gesamtSum += gesamt; fehlerSum += fehler;
  }

  var qGes = gesamtSum > 0 ? (fehlerSum / gesamtSum) * 100 : 0;

  // Text zusammenstellen (mit gewünschter Überschrift "Zwischenreport")
  var txt =
    'Zwischenreport\n'
    + kwStr + '\n\n'
    + 'Kunde: ' + kundenname + '\n'
    + 'Kundennummer: ' + kundennr + '\n'
    + 'Auftragsnummer: ' + auftrag + '\n'
    + 'Anwesenheitstage: ' + checkedDaysText + '\n'
    + personsText
    + 'Summe: Prüfungen: ' + gesamtSum + ', Nicht bestanden: ' + fehlerSum + ', Fehlerquote: ' + qGes.toFixed(2) + ' %';

  return txt;
}
function copyInterimReport(card) {
  var successEl = card.querySelector('.interim-copy-success');
  function showInlineSuccess(){
    if (successEl) { successEl.classList.remove('d-none'); setTimeout(function(){ successEl.classList.add('d-none'); }, 2000); }
  }
  try {
    var text = buildInterimReportForCard(card);
    if (navigator.clipboard && navigator.clipboard.writeText) {
      navigator.clipboard.writeText(text).then(function(){ showInlineSuccess(); }, function(){
        fallbackCopyAll(text, showInlineSuccess);
      });
    } else {
      fallbackCopyAll(text, showInlineSuccess);
    }
  } catch(err) {
    alert(err.message || 'Zwischenreport konnte nicht erstellt werden.');
  }
}

/* ===== Start ===== */
(function init(){
  var payload = loadAllFromCookies();
  buildUIFromData(payload);
})();
</script>

</body>
</html>
