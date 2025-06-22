# Webertechsalesportal.github.io
index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>WeberTech Sales Portal</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      margin: 0;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .filter {
      text-align: center;
      margin-bottom: 20px;
    }
    .filter input {
      padding: 5px 10px;
      margin: 5px;
    }
    table {
      border-collapse: collapse;
      width: 100%;
      background: #fff;
      box-shadow: 0 0 10px #ccc;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 10px;
      text-align: center;
    }
    th {
      background: #333;
      color: white;
    }
    .export {
      margin-top: 10px;
      text-align: center;
    }
    button {
      padding: 10px 15px;
      background: #007bff;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background: #0056b3;
    }
    @media (max-width: 600px) {
      table, thead, tbody, th, td, tr {
        display: block;
      }
      th {
        position: sticky;
        top: 0;
        background: #222;
        color: white;
      }
      td {
        text-align: right;
        padding-left: 50%;
        position: relative;
      }
      td::before {
        content: attr(data-label);
        position: absolute;
        left: 0;
        width: 50%;
        padding-left: 10px;
        font-weight: bold;
        text-align: left;
      }
    }
  </style>
</head>
<body>

  <h1>WeberTech Sales Portal</h1>

  <div class="filter">
    <label for="startDate">From: </label>
    <input type="date" id="startDate">
    <label for="endDate">To: </label>
    <input type="date" id="endDate">
    <button onclick="filterByDate()">Filter</button>
  </div>

  <table id="salesTable">
    <thead>
      <tr>
        <th>Date</th>
        <th>Customer</th>
        <th>Product</th>
        <th>Amount</th>
      </tr>
    </thead>
    <tbody>
      <!-- Sample daily-separated data -->
      <tr><td>2025-06-20</td><td>Alice</td><td>Router</td><td>₦25,000</td></tr>
      <tr><td>2025-06-20</td><td>Bob</td><td>Switch</td><td>₦15,000</td></tr>
      <tr><td>2025-06-21</td><td>Chuka</td><td>CCTV</td><td>₦45,000</td></tr>
      <tr><td>2025-06-22</td><td>Dami</td><td>Inverter</td><td>₦120,000</td></tr>
      <tr><td>2025-06-22</td><td>Esther</td><td>UPS</td><td>₦30,000</td></tr>
    </tbody>
  </table>

  <div class="export">
    <button onclick="exportToExcel()">Export to Excel</button>
  </div>

  <script>
    function filterByDate() {
      const start = document.getElementById("startDate").value;
      const end = document.getElementById("endDate").value;
      const rows = document.querySelectorAll("#salesTable tbody tr");

      rows.forEach(row => {
        const date = row.children[0].textContent;
        if ((!start || date >= start) && (!end || date <= end)) {
          row.style.display = "";
        } else {
          row.style.display = "none";
        }
      });
    }

    function exportToExcel() {
      let table = document.getElementById("salesTable");
      let html = table.outerHTML;
      let url = 'data:application/vnd.ms-excel,' + escape(html);
      let link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "sales_data.xls");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }
  </script>

</body>
</html>
