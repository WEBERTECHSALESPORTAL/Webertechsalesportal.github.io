<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WEBERTECH SALES PORTAL</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
  <style>
    body { font-family: Arial; background:#f0f0f0; padding:20px; max-width:1000px; margin:auto; }
    .card { background:#fff; border-radius:8px; padding:20px; margin-bottom:20px; box-shadow:0 2px 6px rgba(0,0,0,0.1); }
    input, select, button { padding:8px; margin:8px 0; width:100%; max-width:260px; }
    button { cursor:pointer; background:#007BFF; color:#fff; border:none; border-radius:4px; }
    button.logout { background:#dc3545; float:right; }
    table { width:100%; border-collapse:collapse; margin-top:20px; }
    th, td { border:1px solid #ccc; padding:8px; text-align:left; }
    th { background:#eee; }
    .flex { display:flex; gap:20px; flex-wrap:wrap; margin-bottom:15px; }
    .error { color:red; margin-top:10px; }
    #dashboard, #adminPanel, #logoutBtn { display:none; }
  </style>
</head>
<body>

  <div class="card" style="text-align:center;">
    <img src="https://i.ibb.co/SXbQMjqc/IMG-20250614-WA0016.jpg" width="120" style="margin:auto;">
    <h1>WEBERTECH SALES PORTAL</h1>
  </div>

  <div id="loginPage" class="card">
    <h3>Login</h3>
    <input id="username" placeholder="Username or email">
    <input id="password" type="password" placeholder="Password">
    <button onclick="login()">Login</button>
    <div id="loginError" class="error"></div>
  </div>

  <div id="dashboard" class="card">
    <button id="logoutBtn" class="logout" onclick="logout()">Logout</button>
    <h2>Welcome, <span id="empName"></span></h2>

    <div id="adminPanel" class="card">
      <h3>Manage Stock</h3>
      <div class="flex">
        <select id="stockCategory">
          <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
          <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
        </select>
        <input id="stockItem" placeholder="New stock item">
        <input id="stockQty" type="number" placeholder="Qty">
        <button onclick="addStock()">Add Stock</button>
      </div>
      <table id="stockTable"><thead><tr><th>Category</th><th>Item</th><th>Qty</th></tr></thead><tbody></tbody></table>
    </div>

    <h3>Record Sale</h3>
    <div class="flex">
      <select id="saleCategory">
        <option>Cyber Services</option><option>Stationery</option><option>Electronics</option>
        <option>Gas Refill</option><option>Full Gas Cylinder</option><option>Gas Accessories</option>
      </select>
      <input id="saleItem" placeholder="Item or Service">
      <input id="saleQty" type="number" placeholder="Qty">
      <input id="salePrice" type="number" placeholder="Unit Price (KES)">
      <select id="salePayment">
        <option>Cash</option><option>MPESA Till</option><option>Bank Paybill</option><option>Send Money</option>
      </select>
      <input id="saleMpesa" placeholder="MPESA Confirmation ID (optional)">
      <button onclick="addSale()">Add Sale</button>
    </div>

    <div class="flex">
      <h4>Total Sales: KES <span id="totalSales">0</span></h4>
      <h4>Daily Wage: KES <span id="dailyWage">0</span></h4>
    </div>

    <div class="flex">
      <label>From: <input type="date" id="fromDate" onchange="renderSales()"></label>
      <label>To: <input type="date" id="toDate" onchange="renderSales()"></label>
      <input id="search" placeholder="Search..." oninput="renderSales()">
      <button onclick="print()">Print</button>
      <button onclick="exportToExcel()">Export to Excel</button>
    </div>

    <table id="salesTable"><thead><tr>
      <th>Date</th><th>Employee</th><th>Category</th><th>Item</th><th>Qty</th><th>Price</th><th>Total</th><th>Payment</th><th>MPESA</th><th>Action</th>
    </tr></thead><tbody></tbody></table>

    <canvas id="chartCanvas" style="margin-top:30px;"></canvas>
  </div>

<script>
  const users = {
    "fkioko@webergroup":{name:"Fidelis Kioko",password:"fidelis@weber",isAdmin:true},
    "fmwangi@webergroup":{name:"Florence Mwangi",password:"florence@weber",isAdmin:false},
    "mkyalo@webergroup":{name:"Martin Kyalo",password:"martin@weber",isAdmin:false},
    "powen@webergroup":{name:"Philip Owen",password:"philip@weber",isAdmin:false},
    "dkipngeno@webergroup":{name:"Duncan Kipngeno",password:"duncan@weber",isAdmin:false}
  };

  let currentUser=null,sales=[],stock=[],chart;

  function login(){
    const u=username.value.trim().toLowerCase(),p=password.value;
    if(users[u]&&users[u].password===p){
      currentUser={username:u,name:users[u].name,isAdmin:users[u].isAdmin};
      loginPage.style.display="none";
      dashboard.style.display="block";
      logoutBtn.style.display="inline-block";
      empName.textContent=currentUser.name;
      adminPanel.style.display=currentUser.isAdmin?"block":"none";
      renderStock();renderSales();
    }else loginError.textContent="Invalid username or password";
  }

  function logout(){
    currentUser=null;
    dashboard.style.display="none";
    loginPage.style.display="block";
    loginError.textContent="";
    username.value=password.value="";
    if(chart){ chart.destroy(); chart=null; }
  }

  function addStock(){
    const cat=stockCategory.value,it=stockItem.value.trim(),qt=parseInt(stockQty.value);
    if(!it||qt<=0)return alert("Invalid stock");
    stock.push({category:cat,item:it,quantity:qt});
    stockItem.value=stockQty.value="";
    renderStock();
  }

  function renderStock(){
    const tb=stockTable.querySelector("tbody");
    tb.innerHTML=stock.map(s=>`<tr><td>${s.category}</td><td>${s.item}</td><td>${s.quantity}</td></tr>`).join("");
  }

  function addSale(){
    const cat=saleCategory.value,it=saleItem.value.trim(),qt=parseInt(saleQty.value),
          pr=parseFloat(salePrice.value),pm=salePayment.value,mp=saleMpesa.value.trim();
    if(!it||qt<=0||pr<=0)return alert("Fill valid sale details");
    const st=stock.find(s=>s.item.toLowerCase()===it.toLowerCase());
    if(st){
      if(st.quantity<qt)return alert("Not enough stock");
      st.quantity-=qt;
      renderStock();
    }
    sales.push({date:new Date(),employee:currentUser.name,username:currentUser.username,
      category:cat,item:it,qty:qt,price:pr,total:qt*pr,payment:pm,mpesa:mp});
    saleItem.value=saleQty.value=salePrice.value=saleMpesa.value="";
    renderSales();
  }

  function renderSales(){
    const from=fromDate.value?new Date(fromDate.value):null,to=toDate.value?new Date(toDate.value):null,
          term=search.value.trim().toLowerCase();
    const filtered=sales.filter(s=>{
      if(!currentUser.isAdmin && s.username!==currentUser.username) return false;
      const d=new Date(s.date);
      if(from&&d<from||to&&d>to)return false;
      return !term||Object.values(s).some(v=>String(v).toLowerCase().includes(term));
    });
    let total=0;
    salesTable.querySelector("tbody").innerHTML=filtered.map((s,i)=>{
      total+=s.total;
      return `<tr>
        <td>${new Date(s.date).toLocaleString()}</td><td>${s.employee}</td><td>${s.category}</td><td>${s.item}</td><td>${s.qty}</td><td>${s.price}</td><td>${s.total}</td><td>${s.payment}</td><td>${s.mpesa}</td><td><button onclick="deleteSale(${i})">Delete</button></td>
      </tr>`;
    }).join("");
    totalSales.textContent=total.toFixed(2);
    dailyWage.textContent=total>=1300?300:total>=800?250:total>=650?150:total>=350?100:0;
    renderChart(filtered);
  }

  function deleteSale(i){ sales.splice(i,1); renderSales(); }

  function exportToExcel(){
    const X = sales.filter(s=>currentUser.isAdmin||s.username===currentUser.username)
      .map(s=>({Date:s.date.toLocaleString(),Employee:s.employee,Category:s.category,Item:s.item,Qty:s.qty,Price:s.price,Total:s.total,Payment:s.payment,MPESA:s.mpesa}));
    const ws=XLSX.utils.json_to_sheet(X), wb=XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Sales"); XLSX.writeFile(wb, "webertech_sales.xlsx");
  }

  function renderChart(list){
    if(chart){ chart.destroy(); chart=null; }
    const daily={}, empTotals={};
    list.forEach(s=>{
      const d=s.date.toISOString().split("T")[0];
      daily[d]=(daily[d]||0)+s.total;
      if(currentUser.isAdmin){
        empTotals[s.employee]=empTotals[s.employee]||{};
        empTotals[s.employee][d]=(empTotals[s.employee][d]||0)+s.total;
      }
    });
    const labels=Object.keys(daily).sort();
    const datasets=[{ label: currentUser.isAdmin?"Total Sales":"Your Sales", data:labels.map(d=>daily[d]), borderColor:"#007BFF", fill:false }];
    if(currentUser.isAdmin){
      Object.keys(empTotals).forEach((emp,i)=>{
        const color=["#e6194B","#3cb44b","#ffe119","#4363d8","#f58231"][i%5];
        datasets.push({ label:emp, data:labels.map(d=>empTotals[emp][d]||0), borderColor:color, fill:false });
      });
    }
    const ctx=document.getElementById("chartCanvas").getContext("2d");
    chart=new Chart(ctx,{type:"line",data:{labels,datasets},options:{scales:{y:{beginAtZero:true}}}});
  }

  function print(){ window.print(); }
</script>

</body>
</html>
