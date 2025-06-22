
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>WEBERTECH SALES PORTAL</title>
  <!-- Chart.js for dashboard -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- SheetJS for Excel export -->
  <script src="https://cdn.sheetjs.com/xlsx-0.20.0/package/dist/xlsx.full.min.js"></script>
  <style>
    body { font-family: Arial; background: #f0f0f0; padding: 20px; max-width:1200px; margin:auto; }
    input, select, button { padding: 8px; margin: 5px 0; width: 200px; }
    button { cursor: pointer; }
    .flex { display: flex; gap:20px; flex-wrap:wrap; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 6px; text-align: left; }
    h1, h2, h3 { text-align: center; }
    .admin { background: #eef; padding: 10px; margin-top: 20px; }
    #filterSection input { width:auto; }
    canvas { max-width:100%; margin-top:20px; }
  </style>
</head>
<body>

  <div style="text-align:center;margin-bottom:20px;">
    <img src="https://i.ibb.co/SXbQMjqc/IMG-20250614-WA0016.jpg" width="120" />
    <h1>WEBERTECH SALES PORTAL</h1>
  </div>

  <div id="loginPage">
    <h3>Login</h3>
    <input id="username" placeholder="Username/email" />
    <input id="password" placeholder="Password" type="password" />
    <button onclick="login()">Login</button>
  </div>

  <div id="dashboard" style="display:none;">
    <h2>Welcome, <span id="empName"></span></h2>

    <div id="adminPanel" class="admin" style="display:none;">
      <h3>Admin Panel – Manage Stock</h3>
      <div class="flex">
        <input id="stockItem" placeholder="Stock Item" />
        <input id="stockQty" type="number" placeholder="Quantity" />
        <select id="stockCategory">
          <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
          <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
        </select>
        <button onclick="addStock()">Add Stock</button>
      </div>
      <table id="stockTable"><thead><tr><th>Category</th><th>Item</th><th>Qty</th><th>Actions</th></tr></thead><tbody></tbody></table>
    </div>

    <h3>Record Sale</h3>
    <div class="flex">
      <select id="category"><option>Cyber Services</option><option>Stationery</option>
        <option>Electronics</option><option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
      </select>
      <input id="item" placeholder="Item/Service" />
      <input id="quantity" type="number" placeholder="Quantity" />
      <input id="salesAmount" type="number" placeholder="Unit Price (KES)" />
      <button onclick="recordSale()">Add Sale</button>
    </div>

    <div id="totals" class="flex">
      <h4>Total Sales: KES <span id="totalSales">0</span></h4>
      <h4>Daily Wage: KES <span id="dailyWage">0</span></h4>
    </div>

    <div id="filterSection" class="flex">
      <label>From: <input type="date" id="fromDate" onchange="renderSales()"/></label>
      <label>To: <input type="date" id="toDate" onchange="renderSales()"/></label>
      <input id="search" placeholder="Search sales..." onkeyup="renderSales()" />
      <button onclick="printSales()">Print</button>
      <button onclick="exportToExcel()">Export to Excel</button>
    </div>

    <table id="salesTable"><thead><tr>
      <th>Date</th><th>Employee</th><th>Category</th><th>Item</th><th>Qty</th>
      <th>Price</th><th>Total</th><th>Actions</th>
    </tr></thead><tbody></tbody></table>

    <canvas id="dailySalesChart"></canvas>
  </div>

  <script>
    const employees = {
      "fkioko@webergroup":{name:"Fidelis Kioko", password:"fidelis@weber", isAdmin:true},
      "fmwangi@webergroup":{name:"Florence Mwangi", password:"florence@weber"},
      "mkyalo@webergroup":{name:"Martin Kyalo", password:"martin@weber"},
      "powen@webergroup":{name:"Philip Owen", password:"philip@weber"},
      "dkipngeno@webergroup":{name:"Duncan Kipngeno", password:"duncan@weber"}
    };
    let currentUser=null, salesData=[], stockData=[], salesChart;

    // Load stored data
    window.onload = () => {
      salesData = JSON.parse(localStorage.getItem("salesData")||"[]");
      stockData = JSON.parse(localStorage.getItem("stockData")||"[]");
    };

    function login(){
      const u = document.getElementById("username").value.toLowerCase().trim();
      const p = document.getElementById("password").value;
      if(employees[u] && employees[u].password===p){
        currentUser={...employees[u],username:u};
        loginPage.style.display="none"; dashboard.style.display="block";
        empName.textContent=currentUser.name;
        if(currentUser.isAdmin) adminPanel.style.display="block";
        renderStock(); renderSales();
      } else alert("Invalid credentials");
    }

    function recordSale(){
      const i=item.value, q=parseInt(quantity.value), p=parseFloat(salesAmount.value), cat=category.value;
      if(!i||!q||!p) return alert("Fill all fields");
      salesData.push({date:new Date().toLocaleString(),employee:currentUser.name,username:currentUser.username,category:cat,item:i,qty:q,price:p,total:q*p});
      localStorage.setItem("salesData", JSON.stringify(salesData));
      renderSales();
      clearSaleInputs();
    }
    function clearSaleInputs(){
      [item,quantity,salesAmount].forEach(el=>el.value="");
    }

    function renderSales(){
      const tbody=salesTable.querySelector("tbody");
      tbody.innerHTML="";
      const from=fromDate.value?new Date(fromDate.value):null;
      const to=toDate.value?new Date(toDate.value):null;
      let total=0;

      salesData.filter(s=>{
        if(!currentUser.isAdmin && s.username!==currentUser.username) return false;
        const dn=new Date(s.date);
        if(from && dn<from) return false;
        if(to && dn>to) return false;
        const term=search.value.toLowerCase();
        return term==="" || Object.values(s).some(v=>String(v).toLowerCase().includes(term));
      }).forEach((s,i)=>{
        total += s.total;
        const r=tbody.insertRow();
        ["date","employee","category","item","qty","price","total"].forEach(f=> {
          const c=r.insertCell(); c.textContent=s[f];
        });
        const act=r.insertCell();
        act.innerHTML=`<button onclick="editSale(${i})">Edit</button><button onclick="deleteSale(${i})">Delete</button>`;
      });
      totalSales.textContent=total.toFixed(2);
      calculateWage();
      renderChart();
    }

    function editSale(i){
      const s=salesData[i];
      if(!currentUser.isAdmin && s.username!==currentUser.username) return alert("Access denied");
      const nq=+prompt("Qty:",s.qty), np=+prompt("Price:",s.price);
      if(!nq||!np) return;
      s.qty=nq; s.price=np; s.total=nq*np;
      localStorage.setItem("salesData", JSON.stringify(salesData));
      renderSales();
    }
    function deleteSale(i){
      const s=salesData[i];
      if(!currentUser.isAdmin && s.username!==currentUser.username) return alert("Access denied");
      if(confirm("Delete this sale?")) {
        salesData.splice(i,1);
        localStorage.setItem("salesData", JSON.stringify(salesData));
        renderSales();
      }
    }

    function calculateWage(){
      const tsp = salesData.filter(s=>s.username===currentUser.username).reduce((a,b)=>a+b.total,0);
      let w=0;
      if(tsp>=1300) w=300; else if(tsp>=800) w=250;
      else if(tsp>=650) w=150; else if(tsp>=350) w=100;
      dailyWage.textContent=w;
    }

    function addStock(){
      const i=stockItem.value, q=parseInt(stockQty.value), c=stockCategory.value;
      if(!i||!q) return alert("Enter all stock fields");
      stockData.push({category:c,item:i,quantity:q});
      localStorage.setItem("stockData", JSON.stringify(stockData));
      renderStock(); stockItem.value=""; stockQty.value="";
    }
    function renderStock(){
      if(!currentUser.isAdmin) return;
      const tbody=stockTable.querySelector("tbody");
      tbody.innerHTML="";
      stockData.forEach((s,i)=>{
        const r=tbody.insertRow();
        [s.category,s.item,s.quantity].forEach(v=>r.insertCell().textContent=v);
        const act=r.insertCell();
        act.innerHTML=`<button onclick="editStock(${i})">Edit</button><button onclick="deleteStock(${i})">Delete</button>`;
      });
    }
    function editStock(i){
      const s=stockData[i], nq=+prompt("New quantity",s.quantity);
      if(!nq) return;
      s.quantity=nq;
      localStorage.setItem("stockData", JSON.stringify(stockData));
      renderStock();
    }
    function deleteStock(i){
      if(confirm("Delete this stock item?")){
        stockData.splice(i,1);
        localStorage.setItem("stockData", JSON.stringify(stockData));
        renderStock();
      }
    }

    function renderChart(){
      const daily = {};
      salesData.filter(s=>currentUser.isAdmin ? true : s.username===currentUser.username)
        .forEach(s=>{
        const d = s.date.split(',')[0];
        daily[d]=(daily[d]||0)+s.total;
      });
      const labels = Object.keys(daily), data = Object.values(daily);
      const ctx = document.getElementById('dailySalesChart').getContext('2d');
      if(salesChart) salesChart.destroy();
      salesChart = new Chart(ctx,{type:'bar', data:{labels,datasets:[{label:'Daily Sales (KES)',data,backgroundColor:'rgba(75,192,192,0.7)'}]},options:{scales:{y:{beginAtZero:true}}}});
    }

    function printSales(){ window.print(); }

    function exportToExcel(){
      const filtered = salesData.filter(s=>currentUser.isAdmin ? true : s.username===currentUser.username);
      const data = filtered.map(({date,employee,category,item,qty,price,total})=>({date,employee,category,item,qty,price,total}));
      const ws = XLSX.utils.json_to_sheet(data);
      const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Sales");
      XLSX.writeFile(wb, "webertech_sales.xlsx");
    }
  </script>
</body>
</html>
