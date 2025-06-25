<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" /><meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>WEBERTECH SALES PORTAL</title>
  <!-- Chart.js & SheetJS -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.sheetjs.com/xlsx-0.20.0/dist/xlsx.full.min.js"></script>
  <style>
    body { font-family: Arial; background:#f0f0f0; padding:20px; max-width:1200px; margin:auto; }
    .card { background:#fff; padding:20px; border-radius:8px; margin-bottom:20px; box-shadow:0 2px 8px rgba(0,0,0,0.1); }
    h1,h2,h3,h4 { text-align:center; }
    input, select, button { padding:8px; margin:5px 0; width:100%; max-width:250px; }
    button { background:#007BFF; color:#fff; border:none; cursor:pointer; border-radius:4px; }
    .flex { display:flex; gap:20px; flex-wrap:wrap; }
    table { width:100%; border-collapse:collapse; margin-top:10px; }
    th, td { border:1px solid #ccc; padding:6px; background:#fff; }
    canvas { max-width:100%; margin-top:20px; }
    #logoutBtn { float:right; background:#dc3545; }
    #summaryPanel { display:none; }
    .summary-card { flex:1; text-align:center; padding:10px; border-radius:6px; }
  </style>
</head>
<body>
  <div class="card center">
    <img src="https://i.ibb.co/SXbQMjqc/IMG-20250614-WA0016.jpg" width="120" style="display:block; margin:0 auto;"/>
    <h1>WEBERTECH SALES PORTAL</h1>
  </div>

  <div id="loginPage" class="card">
    <h3>Login</h3>
    <input id="username" placeholder="Username/email"/><br/>
    <input id="password" type="password" placeholder="Password"/><br/>
    <button onclick="login()">Login</button>
    <p id="loginError" style="color:red;"></p>
    <p>Don't have an account? <a href="#" onclick="showRegister()">Register</a></p>
  </div>

  <div id="registerPage" class="card" style="display:none;">
    <h3>Register</h3>
    <input id="regName" placeholder="Full Name"/><br/>
    <input id="regUser" placeholder="Username/email"/><br/>
    <input id="regPass" type="password" placeholder="Password"/><br/>
    <label><input type="checkbox" id="regAdmin"/> Admin?</label><br/>
    <button onclick="register()">Register</button>
    <button onclick="showLogin()">Back to Login</button>
    <p id="regError" style="color:red;"></p>
  </div>

  <div id="dashboard" class="card" style="display:none;">
    <h2>Welcome, <span id="empName"></span></h2>
    <button id="logoutBtn" onclick="logout()">Logout</button>

    <!-- Admin Summary -->
    <div id="summaryPanel" class="flex">
      <div class="summary-card" style="background:#e0f7fa;"><h4>Total Sales</h4><div id="totalSalesAll">KES 0</div></div>
      <div class="summary-card" style="background:#f1f8e9;"><h4>Today's Sales</h4><div id="todaySales">KES 0</div></div>
      <div class="summary-card" style="background:#fff3e0;"><h4>Top Item</h4><div id="topItem">-</div></div>
    </div>

    <!-- Admin Stock -->
    <div id="adminPanel" style="display:none; margin-top:20px;">
      <h3>Manage Stock</h3>
      <div class="flex">
        <select id="stockCat">
          <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
          <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
        </select>
        <input id="stockItem" placeholder="Item"/><input id="stockQty" type="number" placeholder="Qty"/>
        <button onclick="addStock()">Add Stock</button>
      </div>
      <table id="stockTable"><thead><tr><th>Category</th><th>Item</th><th>Qty</th><th>Action</th></tr></thead><tbody></tbody></table>
    </div>

    <!-- Record Sale -->
    <h3>Record Sale</h3>
    <div class="flex">
      <select id="category">
        <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
        <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
      </select>
      <input id="item" placeholder="Item/Service"/>
      <input id="quantity" type="number" placeholder="Qty"/>
      <input id="salesPrice" type="number" placeholder="Unit Price (KES)"/>
      <select id="paymentMethod">
        <option>Cash</option><option>MPESA Till</option><option>Bank Paybill</option><option>Send Money</option>
      </select>
      <input id="mpesaId" placeholder="Confirmation ID (optional)"/>
      <button onclick="recordSale()">Add Sale</button>
    </div>

    <!-- Totals & Filters -->
    <div class="flex">
      <h4>Total Sales: KES <span id="totalSales">0</span></h4>
      <h4>Daily Wage: KES <span id="dailyWage">0</span></h4>
    </div>
    <div class="flex">
      <label>From: <input type="date" id="fromDate" onchange="renderSales()"/></label>
      <label>To: <input type="date" id="toDate" onchange="renderSales()"/></label>
      <input id="search" placeholder="Search…" onkeyup="renderSales()"/>
      <button onclick="printSales()">Print</button>
      <button onclick="exportToExcel()">Export</button>
    </div>

    <!-- Sales Table & Chart -->
    <table id="salesTable"><thead><tr>
      <th>Date</th><th>Employee</th><th>Category</th><th>Item</th><th>Qty</th>
      <th>Price</th><th>Total</th><th>Payment</th><th>MPESA ID</th><th>Action</th>
    </tr></thead><tbody></tbody></table>
    <canvas id="dailyChart"></canvas>
  </div>

  <script>
    let currentUser = null, token="", salesData=[], stockData=[], chart;

    function showRegister(){ loginPage.style.display='none'; registerPage.style.display='block'; }
    function showLogin(){ registerPage.style.display='none'; loginPage.style.display='block'; }

    async function login(){
      const u = username.value.trim(), p = password.value;
      try {
        const r = await fetch("https://your-backend-url/api/login", {
          method:"POST", headers:{"Content-Type":"application/json"},
          body:JSON.stringify({username:u,password:p})
        });
        if (!r.ok) throw new Error();
        const d = await r.json();
        token = d.token; currentUser={username:u,name:d.name,isAdmin:d.isAdmin};
        loginPage.style.display='none'; dashboard.style.display='block';
        empName.textContent = currentUser.name;
        if (currentUser.isAdmin){ adminPanel.style.display='block'; summaryPanel.style.display='flex'; fetchStock(); }
        fetchSales();
      } catch {
        loginError.textContent = "Invalid credentials!";
      }
    }

    async function register(){
      const n=regName.value.trim(), u=regUser.value.trim(), p=regPass.value, a=regAdmin.checked;
      try {
        const r = await fetch("https://your-backend-url/api/register", {
          method:"POST",headers:{"Content-Type":"application/json"},
          body:JSON.stringify({name:n,username:u,password:p,isAdmin:a})
        });
        if (!r.ok) throw new Error(await r.json().error);
        alert("Registered!");
        showLogin();
      } catch(e){ regError.textContent = e.message; }
    }

    function logout(){ currentUser=null;token="";dashboard.style.display='none';loginPage.style.display='block'; }

    async function fetchSales(){
      const res = await fetch("https://your-backend-url/api/sales",{headers:{Authorization:`Bearer ${token}`}});
      salesData = await res.json(); renderSales(); renderChart(); renderSummary();
    }

    async function fetchStock(){
      const res = await fetch("https://your-backend-url/api/stock",{headers:{Authorization:`Bearer ${token}`}});
      stockData = await res.json(); renderStock();
    }

    async function addStock(){
      const cat=stockCat.value, i=stockItem.value.trim(), q=+stockQty.value;
      if(!i||q<=0) return alert("Invalid input");
      await fetch("https://your-backend-url/api/stock", {
        method:"POST",headers:{"Content-Type":"application/json", Authorization:`Bearer ${token}`},
        body:JSON.stringify({category:cat,item:i,quantity:q})
      });
      fetchStock();
    }

    function renderStock(){
      stockTable.querySelector("tbody").innerHTML = stockData.map(s => `
        <tr><td>${s.category}</td><td>${s.item}</td><td>${s.quantity}</td><td></td></tr>`).join("");
    }

    async function recordSale(){
      const data = {
        category: category.value,
        item: item.value.trim(),
        qty: +quantity.value,
        price: +salesPrice.value,
        total: +quantity.value * +salesPrice.value,
        paymentMethod: paymentMethod.value,
        mpesaCode: mpesaId.value.trim()
      };
      if(!data.item||data.qty<=0||data.price<=0) return alert("Invalid sale");
      await fetch("https://your-backend-url/api/sales", {
        method:"POST", headers:{"Content-Type":"application/json", Authorization:`Bearer ${token}`},
        body:JSON.stringify(data)
      });
      fetchSales();
      [item,quantity,salesPrice,mpesaId].forEach(el => el.value="");
    }

    function renderSales(){
      const from = fromDate.value?new Date(fromDate.value):null;
      const to = toDate.value?new Date(toDate.value):null;
      const searchTerm = search.value.trim().toLowerCase();
      const tbody = salesTable.querySelector("tbody");
      tbody.innerHTML = salesData.filter(s => {
        const d=new Date(s.date);
        if(from && d<from||to && d>to) return false;
        return searchTerm===""||Object.values(s).some(v=>String(v).toLowerCase().includes(searchTerm));
      }).map((s,i)=>`
        <tr>
          <td>${new Date(s.date).toLocaleString()}</td>
          <td>${s.employee}</td>
          <td>${s.category}</td>
          <td>${s.item}</td>
          <td>${s.qty}</td>
          <td>${s.price}</td>
          <td>${s.total}</td>
          <td>${s.paymentMethod}</td>
          <td>${s.mpesaCode||''}</td>
          <td><button onclick="deleteSale(${i})">Delete</button></td>
        </tr>`
      ).join("");

      const total = salesData.reduce((a,s)=>a+s.total,0);
      const today = salesData.filter(s=>new Date(s.date).toDateString()===new Date().toDateString())
        .reduce((a,s)=>a+s.total,0);

      totalSales.textContent = total.toFixed(2);
      dailyWage.textContent = total>=1300?300:total>=800?250:total>=650?150:total>=350?100:0;

      if(currentUser.isAdmin){
        totalSalesAll.textContent = `KES ${total.toFixed(2)}`;
        todaySales.textContent = `KES ${today.toFixed(2)}`;
      }
    }

    function deleteSale(i){
      fetch(`https://your-backend-url/api/sales`,{method:"DELETE",headers:{Authorization:`Bearer ${token}`, "Content-Type":"application/json"},body:JSON.stringify({id:salesData[i]._id})})
        .then(fetchSales);
    }

    function renderSummary(){
      if(!currentUser.isAdmin) return;
      let topItem="";
      const tally = {};
      salesData.forEach(s=> tally[s.item] = (tally[s.item]||0)+s.total);
      topItem = Object.entries(tally).sort((a,b)=>b[1]-a[1])[0]?.[0]||"-";
      topItemEl.textContent = topItem;
    }

    function renderChart(){
      const tally = {};
      salesData.forEach(s=>{
        const d = new Date(s.date).toLocaleDateString();
        tally[d] = (tally[d]||0) + s.total;
      });
      const ctx = dailyChart.getContext("2d");
      if(chart) chart.destroy();
      chart = new Chart(ctx,{type:"bar",data:{labels:Object.keys(tally),datasets:[{label:"KES Sales",data:Object.values(tally),backgroundColor:"rgba(75,192,192,0.7)"}]},options:{scales:{y:{beginAtZero:true}}}});
    }

    function printSales(){ window.print(); }
    function exportToExcel(){
      const ws = XLSX.utils.json_to_sheet(salesData);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "Sales");
      XLSX.writeFile(wb, "webertech_sales.xlsx");
    }
  </script>
</body>
</html>
