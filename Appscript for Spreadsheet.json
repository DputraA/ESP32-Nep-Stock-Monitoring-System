function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var timestamp = new Date();

  var ip = e.parameter.ip;
  var nama = e.parameter.nama;
  var status = e.parameter.status;

  // ==== Jika ada parameter ip, log ke Sheet1 ====
  if (ip) {
    var sheet1 = ss.getSheets()[0]; // Sheet pertama untuk log IP
    sheet1.insertRowAfter(1);  // Tambah baris baru di bawah header (baris ke-2)
    sheet1.getRange(2, 1).setValue(timestamp);  // Kolom A = Timestamp
    sheet1.getRange(2, 2).setValue(String(ip)); // Kolom B = IP
    return ContentService.createTextOutput("IP Logged");
  }

  // ==== Jika ada parameter nama dan status, update dan log ====
  if (nama && status) {
    var sheet2 = ss.getSheetByName("status"); // Sheet2 untuk status utama
    if (!sheet2) return ContentService.createTextOutput("Sheet 'status' tidak ditemukan");

    var data = sheet2.getDataRange().getValues();
    var found = false;

    for (var i = 1; i < data.length; i++) {
      if (data[i][0] == nama) {
        sheet2.getRange(i + 1, 2).setValue(status);      // Kolom B = Status
        sheet2.getRange(i + 1, 3).setValue(timestamp);   // Kolom C = Timestamp
        found = true;
        break;
      }
    }

    if (!found) {
      sheet2.appendRow([nama, status, timestamp]);
    }

    var sheet3 = ss.getSheetByName("logs"); // Sheet3 untuk log
    if (!sheet3) return ContentService.createTextOutput("Sheet 'logs' tidak ditemukan");

    sheet3.insertRowAfter(1);  // Tambahkan baris baru di atas
    sheet3.getRange(2, 1).setValue(timestamp);
    sheet3.getRange(2, 2).setValue(nama);
    sheet3.getRange(2, 3).setValue(status);

    return ContentService.createTextOutput("Status Updated & Logged");
  }

  // ==== Jika tidak ada parameter: kembalikan IP terbaru dari baris ke-2 ====
  var sheet1 = ss.getSheets()[0];
  if (sheet1.getLastRow() < 2) return ContentService.createTextOutput("No IP Logged Yet");

  // ==== Tambah fitur baru: Get JSON Logs ====
  if (e.parameter.get == 'logs') {
    var sheet3 = ss.getSheetByName("logs");
    if (!sheet3) return ContentService.createTextOutput("Logs sheet not found");
    var rows = sheet3.getRange(2,1, Math.min(10, sheet3.getLastRow()-1), 3).getValues();
    var arr = rows.map(function(r){ return { timestamp: r[0], nama: r[1], status: r[2] }; });
    var out = { logs: arr };
    return ContentService.createTextOutput(JSON.stringify(out))
      .setMimeType(ContentService.MimeType.JSON);
  }
  var latestIP = sheet1.getRange(2, 2).getValue();  // Kolom B, baris ke-2 (IP terbaru)
  return ContentService.createTextOutput(latestIP);
}
