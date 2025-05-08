<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attendance - BCA 2nd Semester</title>
  <style>
    body {
      font-family: Arial;
      padding: 20px;
    }
    table {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
    }
    th, td {
      padding: 10px;
      border: 1px solid #ccc;
    }
    th {
      background-color: #3498db;
      color: white;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      background-color: green;
      color: white;
      border: none;
      cursor: pointer;
      margin-right: 10px;
    }
    select {
      padding: 5px;
      margin-right: 10px;
    }
    .loading {
      display: inline-block;
      width: 20px;
      height: 20px;
      border: 3px solid rgba(0,0,0,.3);
      border-radius: 50%;
      border-top-color: #3498db;
      animation: spin 1s ease-in-out infinite;
      margin-left: 10px;
    }
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
    .file-input {
      display: none;
    }
  </style>
</head>
<body>

  <h1>Shri Ram Murti Smarak International Business School</h1>

  <label>Subject:
    <select id="subjectSelect">
      <option value="DM">Discrete Mathematics (DM)</option>
      <option value="DE">Digital Electronics (DE)</option>
      <option value="DS">Data Structure (DS)</option>
      <option value="PC">Professional Communication (PC)</option>
      <option value="PP">Python Programming (PP)</option>
      <option value="IS">Information System (IS)</option>
      <option value="ESEP(SS)">ESEP(SS)</option>
      <option value="ESEP(QAAR)">ESEP(QAAR)</option>
      <option value="PSY">Psychology</option>
    </select>
  </label>

  <label>Period:
    <select id="periodSelect">
      <option>8:25-9:25</option>
      <option>9:30-10:25</option>
      <option>10:35-11:25</option>
      <option>11:40-12:40</option>
      <option>12:40-1:10</option>
      <option>1:10-2:10</option>
      <option>2:15-3:15</option>
      <option>3:20-4:20</option>
    </select>
  </label>

  <table>
    <thead>
      <tr>
        <th>Roll No.</th>
        <th>Name</th>
        <th>Present</th>
      </tr>
    </thead>
    <tbody id="studentList"></tbody>
  </table>

  <button onclick="saveAttendance()" id="saveBtn">Save Attendance</button>
  <button onclick="loadAttendance()">Load Attendance</button>
  <button onclick="downloadReport()">Download Monthly Report</button>
  <button onclick="showPercentage()">Show Monthly Percentage</button>

  <input type="file" id="fileInput" class="file-input" accept=".json" onchange="handleFileSelect(event)">
  
  <div id="percentageResult"></div>

  <script>
    const students = [
      "Pranat Singh", "Ritika Singh", "Vardan Gupta", "Tejasav Rana", "Ashish Kannaujiya",
      "Palak Tiwari", "Shilpi Bajpai", "Shlok Awasthi", "Saurang", "Anshul Gupta",
      "Ayush Srivastav", "Vinayak Gupta", "Mahi Yadav", "Tulika Tiwari", "Harshita Dutt Lakheda",
      "Aman Singh", "Sahil Maurya"
    ];

    const tbody = document.getElementById("studentList");
    let attendanceData = {};

    // Initialize student list
    students.forEach((name, index) => {
      const row = document.createElement("tr");

      const rollTd = document.createElement("td");
      rollTd.textContent = index + 1;

      const nameTd = document.createElement("td");
      nameTd.textContent = name;

      const presentTd = document.createElement("td");
      const checkbox = document.createElement("input");
      checkbox.type = "checkbox";
      checkbox.dataset.name = name;
      checkbox.dataset.roll = index + 1;
      presentTd.appendChild(checkbox);

      row.appendChild(rollTd);
      row.appendChild(nameTd);
      row.appendChild(presentTd);
      tbody.appendChild(row);
    });

    // Load any existing data from localStorage
    function loadInitialData() {
      const savedData = localStorage.getItem("attendanceData");
      if (savedData) {
        attendanceData = JSON.parse(savedData);
        console.log("Loaded existing attendance data");
      }
    }
    loadInitialData();

    function saveAttendance() {
      const subject = document.getElementById("subjectSelect").value;
      const period = document.getElementById("periodSelect").value;
      const today = new Date().toLocaleDateString("en-GB"); // dd-mm-yyyy
      const monthKey = today.slice(3, 10); // mm-yyyy
      const saveBtn = document.getElementById("saveBtn");

      // Show loading state
      saveBtn.disabled = true;
      saveBtn.innerHTML = "Saving... <span class='loading'></span>";

      const session = {
        date: today,
        subject: subject,
        period: period,
        present: [],
        absent: []
      };

      document.querySelectorAll("input[type='checkbox']").forEach(cb => {
        const entry = `${cb.dataset.roll}. ${cb.dataset.name}`;
        if (cb.checked) session.present.push(entry);
        else session.absent.push(entry);
      });

      // Add to attendance data
      if (!attendanceData[monthKey]) {
        attendanceData[monthKey] = [];
      }
      attendanceData[monthKey].push(session);

      // Save to localStorage
      localStorage.setItem("attendanceData", JSON.stringify(attendanceData));

      // Create download link
      const dataStr = JSON.stringify(attendanceData, null, 2);
      const blob = new Blob([dataStr], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `attendance_data_${new Date().toISOString().slice(0,10)}.json`;
      a.click();

      // Reset button
      setTimeout(() => {
        saveBtn.disabled = false;
        saveBtn.textContent = "Save Attendance";
        alert("Attendance saved and downloaded as JSON file!");
      }, 500);
    }

    function loadAttendance() {
      document.getElementById('fileInput').click();
    }

    function handleFileSelect(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function(e) {
        try {
          const data = JSON.parse(e.target.result);
          attendanceData = data;
          localStorage.setItem("attendanceData", JSON.stringify(data));
          alert("Attendance data loaded successfully!");
        } catch (error) {
          alert("Error loading file: " + error.message);
        }
      };
      reader.readAsText(file);
    }

    function downloadReport() {
      const month = prompt("Enter month and year (mm-yyyy):");
      if (!month) return;

      if (!attendanceData[month] || attendanceData[month].length === 0) {
        alert("No attendance data found for this month!");
        return;
      }

      let content = `Monthly Attendance Report - ${month}\n\n`;
      attendanceData[month].forEach((session, i) => {
        content += `Session ${i + 1} - Date: ${session.date}, Subject: ${session.subject}, Period: ${session.period}\n`;
        content += `Present: ${session.present.join(", ")}\n`;
        content += `Absent: ${session.absent.join(", ")}\n\n`;
      });

      const blob = new Blob([content], { type: "text/plain" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = `Monthly_Attendance_Report_${month}.txt`;
      link.click();
    }

    function showPercentage() {
      const month = prompt("Enter month and year (mm-yyyy):");
      if (!month) return;

      if (!attendanceData[month] || attendanceData[month].length === 0) {
        alert("No attendance data found for this month!");
        return;
      }

      const count = {};
      students.forEach((name, i) => {
        const key = `${i + 1}. ${name}`;
        count[key] = { present: 0, total: 0 };
      });

      attendanceData[month].forEach(session => {
        students.forEach((name, i) => {
          const key = `${i + 1}. ${name}`;
          if (session.present.includes(key)) {
            count[key].present++;
          }
          count[key].total++;
        });
      });

      let result = `<h3>Monthly Attendance Percentage (${month})</h3><table><tr><th>Roll No.</th><th>Name</th><th>Percentage</th></tr>`;
      for (const key in count) {
        const [roll, ...nameParts] = key.split(" ");
        const percent = count[key].total > 0 
          ? ((count[key].present / count[key].total) * 100).toFixed(2)
          : "0.00";
        result += `<tr><td>${roll}</td><td>${nameParts.join(" ")}</td><td>${percent}%</td></tr>`;
      }
      result += "</table>";
      document.getElementById("percentageResult").innerHTML = result;
    }
  </script>

</body>
</html>
