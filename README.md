<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <title>Hospital Roster Pro</title>
    <style>
        body { font-family: sans-serif; margin: 5px; background: #fff; font-size: 10px; }
        .hospital-header { text-align: center; border-bottom: 2px solid #2c3e50; padding-bottom: 5px; margin-bottom: 10px; }
        .section-header { background: #2c3e50; color: white; padding: 8px; margin-top: 10px; font-weight: bold; display: flex; justify-content: space-between; align-items: center; }
        
        /* Settings Styling */
        .settings-box { background: #f9f9f9; border: 1px dashed #7f8c8d; padding: 10px; margin-bottom: 10px; border-radius: 5px; display: none; text-align: center; }
        .settings-box input { padding: 5px; margin: 5px; border: 1px solid #ccc; width: 200px; font-size: 11px; }

        .controls { background: #eee; padding: 10px; display: flex; gap: 5px; justify-content: center; flex-wrap: wrap; border-radius: 5px; margin-bottom: 10px; }
        button { padding: 8px 12px; border-radius: 4px; border: none; font-weight: bold; color: white; cursor: pointer; font-size: 10px; }
        
        .btn-save { background: #27ae60; }
        .btn-print { background: #e74c3c; }
        .btn-excel { background: #217346; }
        .btn-set { background: #34495e; }
        .btn-add { background: #8e44ad; }
        .btn-del { background: #d35400; font-size: 9px; }

        .add-area { display: flex; align-items: center; gap: 5px; }
        .add-area input { width: 100px !important; background: white !important; height: 22px !important; border: 1px solid #ccc !important; padding: 0 5px; }

        table { width: 100%; border-collapse: collapse; table-layout: fixed; }
        th, td { border: 1px solid #000; text-align: center; padding: 2px; height: 25px; }
        .name-col { width: 130px; text-align: left; font-weight: bold; padding-left: 5px; background: #f9f9f9; }
        .day-row { background: #eee; font-weight: bold; }
        
        /* Shift Colors */
        .sun-bg { background-color: #ffcdd2 !important; }
        .off-bg { background-color: #ffff00 !important; }
        .h-bg { background-color: #27ae60 !important; color: white !important; }
        .a-bg { background-color: #e74c3c !important; color: white !important; }
        
        .total-col { background: #fff9c4; font-weight: bold; width: 35px; color: #d35400; }
        input { width: 100%; border: none; text-align: center; font-weight: bold; font-size: 10px; background: transparent; height: 100%; outline: none; text-transform: uppercase; }

        @media print {
            .controls, .add-area, .settings-box, .btn-set { display: none !important; }
            @page { size: A4 landscape; margin: 5mm; }
            .section-header, .off-bg, .sun-bg, .h-bg, .a-bg { -webkit-print-color-adjust: exact; }
        }
    </style>
</head>
<body>

<div id="mainContainer">
    <div class="hospital-header">
        <h2 id="hName" style="margin: 0; font-size: 16px;">TIBBIA HOSPITAL</h2>
        <p id="hAddr" style="margin: 2px; font-weight: bold;">KAROL BAGH, NEW DELHI</p>
        <p style="margin: 0; font-size: 10px; text-decoration: underline;">OPD & IPD ATTENDANCE ROSTER</p>
    </div>

    <div id="settingsBox" class="settings-box">
        <input type="text" id="editHName" placeholder="Naya Hospital Name">
        <input type="text" id="editHAddr" placeholder="Naya Address">
        <button class="btn-save" onclick="saveSettings()">Update Info</button>
    </div>

    <div class="controls">
        <button class="btn-set" onclick="document.getElementById('settingsBox').style.display='block'">⚙️ NAME/ADDR</button>
        <select id="yearSelect" onchange="refreshAll()"></select>
        <select id="monthSelect" onchange="refreshAll()">
            <option value="0">Jan</option><option value="1">Feb</option><option value="2">Mar</option>
            <option value="3">Apr</option><option value="4">May</option><option value="5">Jun</option>
            <option value="6">Jul</option><option value="7">Aug</option><option value="8">Sep</option>
            <option value="9">Oct</option><option value="10">Nov</option><option value="11">Dec</option>
        </select>
        <button class="btn-save" onclick="saveAllData()">💾 SAVE DATA</button>
        <button class="btn-excel" onclick="exportToExcel()">📊 EXCEL</button>
        <button class="btn-print" onclick="window.print()">🖨️ PRINT</button>
    </div>

    <div class="section-header">
        <span>OPD REGISTRATION</span>
        <div class="add-area">
            <input type="text" id="opdNewName" placeholder="New Name">
            <button class="btn-add" onclick="addNewEmployee('OPD')">ADD</button>
            <select id="opdDelList" style="font-size: 10px;"></select>
            <button class="btn-del" onclick="removeEmployee('OPD')">DEL</button>
        </div>
    </div>
    <table id="opdTable"><thead><tr class="dateRow day-row"></tr><tr class="dayRow day-row"></tr></thead><tbody class="rosterBody"></tbody></table>

    <div class="section-header">
        <span>IPD REGISTRATION</span>
        <div class="add-area">
            <input type="text" id="ipdNewName" placeholder="New Name">
            <button class="btn-add" onclick="addNewEmployee('IPD')">ADD</button>
            <select id="ipdDelList" style="font-size: 10px;"></select>
            <button class="btn-del" onclick="removeEmployee('IPD')">DEL</button>
        </div>
    </div>
    <table id="ipdTable"><thead><tr class="dateRow day-row"></tr><tr class="dayRow day-row"></tr></thead><tbody class="rosterBody"></tbody></table>
</div>

<script>
    let employees = { 'OPD': ["Rinku"], 'IPD': ["Reliever"] };
    const daysArr = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];

    function saveSettings() {
        const n = document.getElementById('editHName').value;
        const a = document.getElementById('editHAddr').value;
        if(n) { document.getElementById('hName').innerText = n.toUpperCase(); localStorage.setItem('hName', n.toUpperCase()); }
        if(a) { document.getElementById('hAddr').innerText = a.toUpperCase(); localStorage.setItem('hAddr', a.toUpperCase()); }
        document.getElementById('settingsBox').style.display = 'none';
    }

    function setupYears() {
        const ys = document.getElementById('yearSelect');
        const cy = new Date().getFullYear();
        for (let y = 2025; y <= 2030; y++) {
            let o = document.createElement('option'); o.value = y; o.innerHTML = y;
            if (y === cy) o.selected = true;
            ys.appendChild(o);
        }
        document.getElementById('monthSelect').value = new Date().getMonth();
        
        // Load data
        const savedN = localStorage.getItem('hName');
        const savedA = localStorage.getItem('hAddr');
        if(savedN) document.getElementById('hName').innerText = savedN;
        if(savedA) document.getElementById('hAddr').innerText = savedA;
        
        const savedE = localStorage.getItem('tibbia-employees');
        if(savedE) employees = JSON.parse(savedE);
    }

    function refreshAll() {
        renderSection('OPD', 'opdTable');
        renderSection('IPD', 'ipdTable');
        updateDelMenu();
        loadAllData();
    }

    function renderSection(dept, tableId) {
        const year = parseInt(document.getElementById('yearSelect').value);
        const month = parseInt(document.getElementById('monthSelect').value);
        const daysInMonth = new Date(year, month + 1, 0).getDate();
        const table = document.getElementById(tableId);
        
        let dateH = '<th class="name-col">DATE</th>';
        let dayH = '<th class="name-col">DAY</th>';
        for (let i = 1; i <= daysInMonth; i++) {
            let d = new Date(year, month, i);
            let dayN = daysArr[d.getDay()];
            let sCls = (dayN === "SUN") ? 'sun-bg' : '';
            dateH += `<th class="${sCls}">${i}</th>`;
            dayH += `<th class="${sCls}">${dayN}</th>`;
        }
        table.querySelector('.dateRow').innerHTML = dateH + '<th class="total-col">TOT</th>';
        table.querySelector('.dayRow').innerHTML = dayH + '<th class="total-col">DUTY</th>';

        let bHtml = '';
        employees[dept].forEach(emp => {
            bHtml += `<tr><td class="name-col">${emp}</td>`;
            for (let i = 1; i <= daysInMonth; i++) {
                let sunCls = (new Date(year, month, i).getDay() === 0) ? 'sun-bg' : '';
                bHtml += `<td class="${sunCls}"><input type="text" oninput="handleInp(this)"></td>`;
            }
            bHtml += `<td class="total-col">0</td></tr>`;
        });
        table.querySelector('.rosterBody').innerHTML = bHtml;
    }

    function handleInp(input) {
        let val = input.value.trim().toUpperCase();
        let td = input.parentElement;
        td.classList.remove('off-bg', 'h-bg', 'a-bg');
        if (val === "OFF") td.classList.add('off-bg');
        else if (val === "H") td.classList.add('h-bg');
        else if (val === "A") td.classList.add('a-bg');
        updateCount(td.parentElement);
    }

    function updateCount(row) {
        let count = 0;
        row.querySelectorAll('input').forEach(i => {
            let v = i.value.trim().toUpperCase();
            if(v !== "" && v !== "OFF" && v !== "A" && v !== "H") {
                count += v.includes("+") ? 2 : 1;
            }
        });
        row.querySelector('.total-col').innerText = count;
    }

    function addNewEmployee(dept) {
        const input = document.getElementById(dept.toLowerCase() + 'NewName');
        const name = input.value.trim();
        if (name) {
            employees[dept].push(name);
            localStorage.setItem('tibbia-employees', JSON.stringify(employees));
            input.value = '';
            refreshAll();
        }
    }

    function removeEmployee(dept) {
        const val = document.getElementById(dept.toLowerCase() + 'DelList').value;
        if(!val) return;
        if(confirm("Hata dein: " + val + "?")) {
            employees[dept] = employees[dept].filter(e => e !== val);
            localStorage.setItem('tibbia-employees', JSON.stringify(employees));
            refreshAll();
        }
    }

    function updateDelMenu() {
        ['OPD', 'IPD'].forEach(dept => {
            const menu = document.getElementById(dept.toLowerCase() + 'DelList');
            menu.innerHTML = '<option value="">Select</option>';
            employees[dept].forEach(e => {
                let op = document.createElement('option'); op.value = e; op.innerText = e;
                menu.appendChild(op);
            });
        });
    }

    function saveAllData() {
        const y = document.getElementById('yearSelect').value;
        const m = document.getElementById('monthSelect').value;
        const data = {};
        ['OPD', 'IPD'].forEach(dept => {
            data[dept] = {};
            document.querySelectorAll(`#${dept.toLowerCase()}Table .rosterBody tr`).forEach(row => {
                const name = row.querySelector('.name-col').innerText;
                data[dept][name] = Array.from(row.querySelectorAll('input')).map(i => i.value);
            });
        });
        localStorage.setItem(`data-${y}-${m}`, JSON.stringify(data));
        alert("Saved!");
    }

    function loadAllData() {
        const y = document.getElementById('yearSelect').value;
        const m = document.getElementById('monthSelect').value;
        const saved = JSON.parse(localStorage.getItem(`data-${y}-${m}`));
        if (!saved) return;
        ['OPD', 'IPD'].forEach(dept => {
            document.querySelectorAll(`#${dept.toLowerCase()}Table .rosterBody tr`).forEach(row => {
                const name = row.querySelector('.name-col').innerText;
                if (saved[dept] && saved[dept][name]) {
                    const inputs = row.querySelectorAll('input');
                    saved[dept][name].forEach((v, i) => { if(inputs[i]) { inputs[i].value = v; handleInp(inputs[i]); } });
                }
            });
        });
    }

    function exportToExcel() {
        let csv = [];
        const h = document.getElementById('hName').innerText;
        csv.push(h + " ROSTER");
        ['opdTable', 'ipdTable'].forEach(id => {
            document.querySelectorAll(`#${id} tr`).forEach(row => {
                let rd = [];
                row.querySelectorAll('th, td').forEach(col => {
                    let t = col.querySelector('input') ? col.querySelector('input').value : col.innerText;
                    rd.push('"' + t.trim() + '"');
                });
                csv.push(rd.join(','));
            });
            csv.push('\n');
        });
        const blob = new Blob([csv.join('\n')], { type: 'text/csv' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = `${h}_Attendance.csv`;
        link.click();
    }

    window.onload = () => { setupYears(); refreshAll(); };
</script>
</body>
</html>


