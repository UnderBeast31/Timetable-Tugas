<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Timetable Tugas</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap');

    :root {
      --bg-color: #f5f8fc;
      --text-color: #333;
      --header-bg: linear-gradient(90deg, #2b68c4, #00b7ff);
      --card-bg: #ffffff;
    }

    [data-theme="dark"] {
      --bg-color: #1f1f1f;
      --text-color: #f1f1f1;
      --header-bg: linear-gradient(90deg, #222, #444);
      --card-bg: #2b2b2b;
    }

    body {
      font-family: 'Poppins', sans-serif;
      margin: 0;
      padding: 0;
      background-color: var(--bg-color);
      color: var(--text-color);
      transition: background-color 0.3s, color 0.3s;
    }

    header {
      background: var(--header-bg);
      padding: 20px;
      color: white;
      text-align: center;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }

    h1 {
      margin: 0;
      font-size: 28px;
    }

    .top-summary {
      display: flex;
      justify-content: space-around;
      align-items: center;
      padding: 10px 20px;
      background-color: var(--card-bg);
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
      margin: 20px auto;
      border-radius: 10px;
      max-width: 1000px;
    }

    .top-summary div {
      text-align: center;
      font-weight: bold;
    }

    .theme-toggle {
      position: absolute;
      top: 20px;
      right: 20px;
      background-color: transparent;
      border: 2px solid white;
      color: white;
      padding: 5px 10px;
      border-radius: 5px;
      cursor: pointer;
    }

    .form-container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
      padding: 20px;
      background-color: var(--card-bg);
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
      margin: 20px auto;
      border-radius: 10px;
      max-width: 1000px;
    }

    input, select, button {
      padding: 10px;
      border-radius: 5px;
      border: 1px solid #ccc;
      font-size: 14px;
    }

    button {
      background-color: #2b68c4;
      color: white;
      border: none;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #1d4e91;
    }

    table {
      width: 90%;
      margin: 0 auto 40px auto;
      border-collapse: collapse;
    }

    th, td {
      padding: 12px;
      border: 1px solid #ddd;
      text-align: center;
    }

    th {
      background-color: #2b68c4;
      color: white;
    }

    .chart-section {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      align-items: flex-start;
      gap: 20px;
      padding: 20px;
    }

    .chart-box {
      background-color: var(--card-bg);
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      width: 300px;
    }

    .chart-box h3 {
      margin-top: 0;
      text-align: center;
      font-size: 16px;
    }

    canvas {
      margin: auto;
    }

    .progress-info {
      text-align: center;
      margin-top: 10px;
      font-weight: bold;
    }

    .button-group {
      display: flex;
      justify-content: center;
      margin-bottom: 20px;
      gap: 10px;
    }
  </style>
</head>
<body>
  <header>
    <h1>Timetable Tugas Harian</h1>
    <button class="theme-toggle" onclick="toggleTheme()">Toggle Mode</button>
  </header>

  <div class="top-summary">
    <div>Tanggal Hari Ini: <span id="todayDate"></span></div>
    <div>Total Tugas: <span id="totalTasks">0</span></div>
    <div>Waktu Sekarang: <span id="currentTime"></span></div>
  </div>

  <div class="chart-section">
    <div class="chart-box">
      <h3>Tasks by Priority</h3>
      <canvas id="priorityChart" width="280" height="200"></canvas>
    </div>
    <div class="chart-box">
      <h3>Tasks Status</h3>
      <p>Due Today: <span id="dueToday">0</span></p>
      <p>Overdue Tasks: <span id="overdueTasks">0</span></p>
    </div>
    <div class="chart-box">
      <h3>Progress Status</h3>
      <canvas id="statusChart" width="280" height="200"></canvas>
      <div class="progress-info" id="progressSummary"></div>
    </div>
    <div class="chart-box">
      <h3>Tasks by Category</h3>
      <canvas id="categoryChart" width="280" height="200"></canvas>
    </div>
  </div>

  <div class="form-container">
    <input type="date" id="tanggal">
    <input type="time" id="jam">
    <input type="text" id="jenis" placeholder="Jenis Tugas">
    <select id="urgensi">
      <option value="Urgent">Urgent</option>
      <option value="High">High</option>
      <option value="Med">Med</option>
      <option value="Low">Low</option>
    </select>
    <select id="status">
      <option value="Completed">Completed</option>
      <option value="In Progress">In Progress</option>
      <option value="On Hold">On Hold</option>
      <option value="Canceled">Canceled</option>
    </select>
    <button onclick="tambahTugas()">Tambah</button>
  </div>

  <div class="button-group">
    <button onclick="unduhTabelCSV()">Download CSV</button>
    <button onclick="hapusSemuaTugas()">Hapus Semua</button>
  </div>

  <table id="tabelTugas">
    <thead>
      <tr>
        <th>Tanggal</th>
        <th>Jam</th>
        <th>Jenis Tugas</th>
        <th>Tingkat Urgensi</th>
        <th>Status</th>
        <th>Aksi</th>
      </tr>
    </thead>
    <tbody>
      <!-- Baris tugas akan ditambahkan di sini -->
    </tbody>
  </table>

  <script>
    const tugasList = [];

    function updateTodayDate() {
      const today = new Date();
      const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
      document.getElementById('todayDate').textContent = today.toLocaleDateString('id-ID', options);
    }

    function updateCurrentTime() {
      const now = new Date();
      const timeString = now.toLocaleTimeString('id-ID');
      document.getElementById('currentTime').textContent = timeString;
    }

    setInterval(updateCurrentTime, 1000);

    function tambahTugas() {
      const tanggal = document.getElementById('tanggal').value;
      const jam = document.getElementById('jam').value;
      const jenis = document.getElementById('jenis').value;
      const urgensi = document.getElementById('urgensi').value;
      const status = document.getElementById('status').value;

      if (!tanggal || !jam || !jenis) return;

      const tugas = { tanggal, jam, jenis, urgensi, status };
      tugasList.push(tugas);

      renderTable();
      updateCharts();
    }

    function hapusTugas(index) {
      tugasList.splice(index, 1);
      renderTable();
      updateCharts();
    }

    function hapusSemuaTugas() {
      if (confirm('Apakah Anda yakin ingin menghapus semua tugas?')) {
        tugasList.length = 0;
        renderTable();
        updateCharts();
      }
    }

function renderTable() {
  const tbody = document.querySelector('#tabelTugas tbody');
  tbody.innerHTML = '';
  tugasList.forEach((tugas, index) => {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${tugas.tanggal}</td>
      <td>${tugas.jam}</td>
      <td>${tugas.jenis}</td>
      <td>${tugas.urgensi}</td>
      <td>${tugas.status}</td>
      <td>
        <button onclick="editStatus(${index})">Edit</button>
        <button onclick="hapusTugas(${index})">Hapus</button>
      </td>
    `;
    tbody.appendChild(row);
  });
}

    function updateCharts() {
      const priorityCount = { Urgent: 0, High: 0, Med: 0, Low: 0 };
      const statusCount = { 'Completed': 0, 'In Progress': 0, 'On Hold': 0, 'Canceled': 0 };
      const categoryCount = {};
      let dueToday = 0;
      let overdue = 0;
      const today = new Date().toISOString().split('T')[0];

      tugasList.forEach(tugas => {
        priorityCount[tugas.urgensi]++;
        statusCount[tugas.status]++;
        categoryCount[tugas.jenis] = (categoryCount[tugas.jenis] || 0) + 1;

        if (tugas.tanggal === today) dueToday++;
        else if (tugas.tanggal < today && tugas.status !== 'Completed') overdue++;
      });

      document.getElementById('dueToday').textContent = dueToday;
      document.getElementById('overdueTasks').textContent = overdue;
      document.getElementById('totalTasks').textContent = tugasList.length;

      if (window.priorityChartObj) window.priorityChartObj.destroy();
      if (window.statusChartObj) window.statusChartObj.destroy();
      if (window.categoryChartObj) window.categoryChartObj.destroy();

      window.priorityChartObj = new Chart(document.getElementById('priorityChart'), {
        type: 'bar',
        data: {
          labels: Object.keys(priorityCount),
          datasets: [{
            label: 'Prioritas',
            data: Object.values(priorityCount),
            backgroundColor: ['#e74c3c', '#f39c12', '#3498db', '#2ecc71']
          }]
        },
        options: { scales: { y: { beginAtZero: true } } }
      });

      window.statusChartObj = new Chart(document.getElementById('statusChart'), {
        type: 'bar',
        data: {
          labels: Object.keys(statusCount),
          datasets: [{
            label: 'Status',
            data: Object.values(statusCount),
            backgroundColor: ['#27ae60', '#2980b9', '#f1c40f', '#95a5a6']
          }]
        },
        options: { scales: { y: { beginAtZero: true } } }
      });

      window.categoryChartObj = new Chart(document.getElementById('categoryChart'), {
        type: 'doughnut',
        data: {
          labels: Object.keys(categoryCount),
          datasets: [{
            data: Object.values(categoryCount),
            backgroundColor: ['#9b59b6', '#1abc9c', '#e67e22', '#34495e', '#2ecc71']
          }]
        }
      });

      const total = tugasList.length;
      const completed = statusCount['Completed'];
      document.getElementById('progressSummary').textContent = `${completed} dari ${total} tugas telah selesai`;
    }

    function unduhTabelCSV() {
      let csv = 'Tanggal,Jam,Jenis Tugas,Tingkat Urgensi,Status\n';
      tugasList.forEach(tugas => {
        csv += `${tugas.tanggal},${tugas.jam},${tugas.jenis},${tugas.urgensi},${tugas.status}\n`;
      });

      const blob = new Blob([csv], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'tabel_tugas.csv';
      a.click();
      URL.revokeObjectURL(url);
    }

    updateTodayDate();
    updateCurrentTime();

      function toggleTheme() {
      const currentTheme = document.documentElement.getAttribute('data-theme');
      if (currentTheme === 'dark') {
        document.documentElement.setAttribute('data-theme', 'light');
        localStorage.setItem('theme', 'light');
      } else {
        document.documentElement.setAttribute('data-theme', 'dark');
        localStorage.setItem('theme', 'dark');
      }
    }

    // Set initial theme on load
    (function () {
      const savedTheme = localStorage.getItem('theme') || 'light';
      document.documentElement.setAttribute('data-theme', savedTheme);
    })();
    
function editStatus(index) {
  const tbody = document.querySelector('#tabelTugas tbody');
  const row = tbody.children[index];
  const tugas = tugasList[index];

  // Ganti kolom status jadi select dropdown
  row.cells[4].innerHTML = `
    <select id="editStatusSelect${index}">
      <option value="Completed" ${tugas.status === 'Completed' ? 'selected' : ''}>Completed</option>
      <option value="In Progress" ${tugas.status === 'In Progress' ? 'selected' : ''}>In Progress</option>
      <option value="On Hold" ${tugas.status === 'On Hold' ? 'selected' : ''}>On Hold</option>
      <option value="Canceled" ${tugas.status === 'Canceled' ? 'selected' : ''}>Canceled</option>
    </select>
  `;

  // Ganti tombol hapus jadi tombol save
  row.cells[5].innerHTML = `<button onclick="saveStatus(${index})">Save</button>`;
}

function saveStatus(index) {
  const select = document.getElementById(`editStatusSelect${index}`);
  const newStatus = select.value;
  tugasList[index].status = newStatus;

  renderTable();
  updateCharts();
}

  </script>
</body>
</html>
