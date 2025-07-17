# JualanAja.id
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jualin.id</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 2rem;
    }
    h1 {
      color: #2d3748;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }
    th, td {
      padding: 0.5rem;
      border: 1px solid #ccc;
      text-align: left;
    }
    input, select, button {
      margin: 0.5rem 0;
      padding: 0.5rem;
      width: 100%;
    }
    .form-group {
      margin-bottom: 1rem;
    }
    #chartContainer {
      margin-top: 2rem;
    }
    .delete-btn {
      color: red;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h1>📦 Jualin.id</h1>
  <div class="form-group">
    <input type="text" id="productName" placeholder="Nama Produk">
    <input type="date" id="saleDate">
    <input type="number" id="quantity" placeholder="Jumlah Terjual">
    <div style="display: flex; gap: 0.5rem;">
      <select id="currency" style="width: 30%;"></select>
      <input type="number" id="unitPrice" placeholder="Harga Satuan" style="width: 70%;">
    </div>
    <button onclick="addSale()">Tambah Penjualan</button>
    <button onclick="downloadCSV()">Download Data Penjualan (.csv)</button>
  </div>

  <table id="salesTable">
    <thead>
      <tr>
        <th>Produk</th>
        <th>Tanggal</th>
        <th>Jumlah</th>
        <th>Harga Satuan</th>
        <th>Total</th>
        <th>Aksi</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <div id="chartContainer">
    <h2>📈 Grafik Total Penjualan per Produk</h2>
    <canvas id="salesChart" width="400" height="200"></canvas>
  </div>

  <script>
    let chart;
    const productSalesData = {};
    const currencies = [
      "Rp", "$", "€", "£", "¥", "₹", "₽", "₩", "฿", "₫", "₪", "₱", "₴", "₦", "₲",
      "₡", "₵", "₭", "₮", "₼", "₸", "₺", "₨"
    ];

    window.onload = function () {
      const currencySelect = document.getElementById('currency');
      currencies.forEach(code => {
        const option = document.createElement("option");
        option.value = code;
        option.textContent = code;
        currencySelect.appendChild(option);
      });
      document.getElementById('saleDate').valueAsDate = new Date();
    };

    function updateChart() {
      const labels = Object.keys(productSalesData);
      const totals = labels.map(label => productSalesData[label]);
      if (chart) chart.destroy();
      const ctx = document.getElementById('salesChart').getContext('2d');
      chart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Total Penjualan per Produk',
            data: totals,
            backgroundColor: '#38a169'
          }]
        },
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                callback: value => value.toLocaleString('id-ID')
              }
            }
          }
        }
      });
    }

    function addSale() {
      const name = document.getElementById('productName').value;
      const date = document.getElementById('saleDate').value;
      const quantity = parseInt(document.getElementById('quantity').value);
      const price = parseInt(document.getElementById('unitPrice').value);
      const currency = document.getElementById('currency').value;

      if (!name || !date || isNaN(quantity) || isNaN(price)) {
        alert("Semua kolom harus diisi dengan benar.");
        return;
      }

      const total = quantity * price;
      const table = document.getElementById('salesTable').getElementsByTagName('tbody')[0];
      const row = table.insertRow();
      row.insertCell(0).innerText = name;
      row.insertCell(1).innerText = new Date(date).toLocaleDateString('id-ID');
      row.insertCell(2).innerText = quantity;
      row.insertCell(3).innerText = `${currency}${price.toLocaleString('id-ID')}`;
      row.insertCell(4).innerText = `${currency}${total.toLocaleString('id-ID')}`;
      const aksiCell = row.insertCell(5);
      aksiCell.innerHTML = '<span class="delete-btn" onclick="deleteRow(this)">Hapus</span>';

      productSalesData[name] = (productSalesData[name] || 0) + total;
      updateChart();

      document.getElementById('productName').value = "";
      document.getElementById('saleDate').valueAsDate = new Date();
      document.getElementById('quantity').value = "";
      document.getElementById('unitPrice').value = "";
    }

    function deleteRow(el) {
      const row = el.parentNode.parentNode;
      const name = row.cells[0].innerText;
      const totalText = row.cells[4].innerText;
      const total = parseInt(totalText.replace(/[^0-9]/g, ''));
      if (productSalesData[name]) {
        productSalesData[name] -= total;
        if (productSalesData[name] <= 0) delete productSalesData[name];
      }
      row.remove();
      updateChart();
    }

    function downloadCSV() {
      const rows = document.querySelectorAll("#salesTable tr");
      let csvContent = "data:text/csv;charset=utf-8,";
      rows.forEach(row => {
        const cols = row.querySelectorAll("td, th");
        const rowData = Array.from(cols).map(col => col.innerText);
        csvContent += rowData.join(",") + "\n";
      });
      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "data_penjualan_fibrase.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }
  </script>
</body>
</html>
