<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>WEBERTECH SALES PORTAL</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.sheetjs.com/xlsx-0.20.0/package/dist/xlsx.full.min.js"></script>
  <style>
    body { font-family: Arial; background: linear-gradient(to right, #e0eafc, #cfdef3); padding: 20px; max-width:1200px; margin:auto; }
    input, select, button { padding: 8px; margin: 5px 0; width: 200px; border-radius: 4px; border: 1px solid #ccc; }
    button { cursor: pointer; background: #007BFF; color: white; border: none; }
    button:hover { background: #0056b3; }
    .flex { display: flex; gap:20px; flex-wrap:wrap; align-items: center; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
    th, td { border: 1px solid #ccc; padding: 6px; text-align: left; }
    h1, h2, h3 { text-align: center; }
    .admin { background: #eef; padding: 10px; margin-top: 20px; border-radius: 10px; }
    #filterSection input { width:auto; }
    canvas { max-width:100%; margin-top:20px; }
    #logoutBtn { float: right; margin-top: -60px; background: #dc3545; }
    #logoutBtn:hover { background: #a71d2a; }
    .logo-container img { display: block; margin: auto; border-radius: 10px; }
    .suggestion-box { border: 1px solid #ccc; background: #fff; position: absolute; z-index: 1000; max-height: 150px; overflow-y: auto; }
    .suggestion-item { padding: 4px 8px; cursor: pointer; }
    .suggestion-item:hover { background-color: #f0f0f0; }
  </style>
</head>
<body>

  <div class="logo-container">
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
    <button id="logoutBtn" onclick="logout()">Logout</button>
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
      <table id="stockTable">
        <thead><tr><th>Category</th><th>Item</th><th>Qty</th><th>Actions</th></tr></thead><tbody></tbody>
      </table>
    </div>

    <h3>Record Sale</h3>
    <div class="flex" style="position: relative;">
      <select id="category">
        <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
        <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
      </select>
      <input id="item" placeholder="Item/Service" oninput="showSuggestions()" autocomplete="off" />
      <div id="suggestions" class="suggestion-box" style="display:none;"></div>
      <input id="quantity" type="number" placeholder="Quantity" />
      <input id="salesAmount" type="number" placeholder="Unit Price (KES)" />
      <select id="paymentMethod">
        <option value="Cash">Cash</option>
        <option value="MPESA Till">MPESA Till</option>
        <option value="MPESA Paybill">MPESA Paybill</option>
        <option value="MPESA Send Money">MPESA Send Money</option>
        <option value="Bank">Bank</option>
      </select>
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

    <table id="salesTable"><thead>
      <tr><th>Date</th><th>Employee</th><th>Category</th><th>Item</th><th>Qty</th><th>Price</th><th>Total</th><th>Payment</th><th>Actions</th></tr>
    </thead><tbody></tbody></table>

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

  window.onload = () => {
    salesData = JSON.parse(localStorage.getItem("salesData")||"[]");
    stockData = JSON.parse(localStorage.getItem("stockData")||"[]");
  };

  function login(){
    const u = username.value.toLowerCase().trim();
    const p = password.value;
    if(employees[u] && employees[u].password===p){
      currentUser={...employees[u],username:u};
      loginPage.style.display="none";
      dashboard.style.display="block";
      empName.textContent=currentUser.name;
      if(currentUser.isAdmin) adminPanel.style.display="block";
      renderStock(); renderSales();
    } else alert("Invalid credentials");
  }

  function logout(){
    currentUser = null;
    dashboard.style.display = "none";
    loginPage.style.display = "block";
  }

  function recordSale(){
    const i=item.value.trim(), q=parseInt(quantity.value), p=parseFloat(salesAmount.value), cat=category.value, pay=paymentMethod.value;
    if(!i || !q || !p || q <= 0 || p <= 0) return alert("Fill all fields with valid values");
    const sale = {
      date: new Date().toLocaleString(), employee: currentUser.name, username: currentUser.username,
      category: cat, item: i, qty: q, price: p, total: q*p, payment: pay
    };
    salesData.push(sale);
    localStorage.setItem("salesData", JSON.stringify(salesData));

    // Reduce stock
    const stockItem = stockData.find(s => s.item.toLowerCase() === i.toLowerCase());
    if(stockItem && stockItem.quantity >= q){
      stockItem.quantity -= q;
      localStorage.setItem("stockData", JSON.stringify(stockData));
      renderStock();
    }

    renderSales(); clearSaleInputs();
  }

  function clearSaleInputs(){
    [item, quantity, salesAmount].forEach(el=>el.value="");
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
      ["date","employee","category","item","qty","price","total","payment"].forEach(f=>r.insertCell().textContent=s[f]);
      r.insertCell().innerHTML=`<button onclick="editSale(${i})">Edit</button><button onclick="deleteSale(${i})">Delete</button>`;
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
    if(tsp>=1300) w=300;
    else if(tsp>=800) w=250;
    else if(tsp>=650) w=150;
    else if(tsp>=350) w=100;
    dailyWage.textContent=w;
  }

  function addStock(){
    const i=stockItem.value.trim(), q=parseInt(stockQty.value), c=stockCategory.value;
    if(!i || !q || q <= 0) return alert("Enter all stock fields");
    stockData.push({category:c,item:i,quantity:q});
    localStorage.setItem("stockData", JSON.stringify(stockData));
    renderStock();
    stockItem.value=""; stockQty.value="";
  }

  function renderStock(){
    if(!currentUser.isAdmin) return;
    const tbody=stockTable.querySelector("tbody");
    tbody.innerHTML="";
    stockData.forEach((s,i)=>{
      const r=tbody.insertRow();
      [s.category,s.item,s.quantity].forEach(v=>r.insertCell().textContent=v);
      r.insertCell().innerHTML=`<button onclick="editStock(${i})">Edit</button><button onclick="deleteStock(${i})">Delete</button>`;
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
    salesData.filter(s=>currentUser.isAdmin ? true : s.username===currentUser.username).forEach(s=>{
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
    const data = filtered.map(({date,employee,category,item,qty,price,total,payment})=>({date,employee,category,item,qty,price,total,payment}));
    const ws = XLSX.utils.json_to_sheet(data);
    const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Sales");
    XLSX.writeFile(wb, "webertech_sales.xlsx");
  }

  function showSuggestions() {
    const input = item.value.toLowerCase();
    const matches = stockData.filter(s => s.item.toLowerCase().includes(input)).map(s => s.item);
    const suggestionBox = document.getElementById("suggestions");
    if (matches.length === 0 || input.length === 0) return suggestionBox.style.display = "none";
    suggestionBox.innerHTML = matches.map(m => `<div class="suggestion-item" onclick="selectSuggestion('${m}')">${m}</div>`).join('');
    suggestionBox.style.display = "block";
  }

  function selectSuggestion(val) {
    item.value = val;
    document.getElementById("suggestions").style.display = "none";
  }
</script>

</body>
</html>
