<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Excel Row Highlighter + Filter</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
    }
    #excelTable {
      border-collapse: collapse;
      width: 100%;
      margin-top: 20px;
    }
    #excelTable th, #excelTable td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: left;
    }
    .highlight {
      background-color: yellow;
    }
    input[type="text"], input[type="file"], button {
      margin: 10px 0;
      padding: 8px;
      font-size: 16px;
    }
    .hidden {
      display: none;
    }
  </style>
</head>
<body>
  <h2>Upload Excel and Highlight & Filter Matching Rows</h2>
  <input type="file" id="upload" accept=".xlsx, .xls">
  <br>
  <input type="text" id="searchInput" placeholder="Search text (e.g. apple, banana)">
  <button onclick="highlightRows()">Filter & Highlight</button>
  <button onclick="resetTable()">Reset Table</button>
  <table id="excelTable"></table>

  <script>
    let globalData = [];

    document.getElementById('upload').addEventListener('change', function (e) {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function (e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });

        const sheetName = workbook.SheetNames[0];
        const sheet = workbook.Sheets[sheetName];

        globalData = XLSX.utils.sheet_to_json(sheet, { header: 1 });
        renderTable(globalData);
      };
      reader.readAsArrayBuffer(file);
    });

    function renderTable(data) {
      const table = document.getElementById('excelTable');
      table.innerHTML = '';

      data.forEach((row, rowIndex) => {
        const tr = document.createElement('tr');
        row.forEach(cell => {
          const td = document.createElement(rowIndex === 0 ? 'th' : 'td');
          td.textContent = cell;
          tr.appendChild(td);
        });
        table.appendChild(tr);
      });
    }

    function highlightRows() {
      const searchInput = document.getElementById('searchInput').value;
      const searchTerms = searchInput.split(',').map(term => term.trim().toLowerCase()).filter(Boolean);

      const table = document.getElementById('excelTable');
      [...table.rows].forEach((row, index) => {
        if (index === 0) {
          row.classList.remove('highlight');
          row.classList.remove('hidden');
          return;
        }

        const rowText = [...row.cells].map(cell => cell.textContent.toLowerCase()).join(' ');
        const match = searchTerms.some(term => rowText.includes(term));

        if (match) {
          row.classList.add('highlight');
          row.classList.remove('hidden');
        } else {
          row.classList.remove('highlight');
          row.classList.add('hidden');
        }
      });
    }

    function resetTable() {
      const table = document.getElementById('excelTable');
      [...table.rows].forEach((row, index) => {
        row.classList.remove('highlight');
        row.classList.remove('hidden');
      });
      document.getElementById('searchInput').value = '';
    }
  </script>
</body>
</html>
