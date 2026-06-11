# 3-Gunden-Eski-Klasor-Temizle-Raporlu
Bu script ile seçtiğiniz klasörü tarayarak 3 günden eski dosyaları silebilirsiniz. 

# Scriptin imzasız olarak çalışması için Execution Policy'i geçici olarak ayarlar.
# Bu komut sadece bu script oturumu için geçerlidir.
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force

$Path = "C:\AI_RecycleBin"
$Days = 3

# Raporlama klasörünü oluşturur.
$ReportPath = "C:\SilinmeRaporlari"
if (-not (Test-Path -Path $ReportPath)) {
    New-Item -ItemType Directory -Path $ReportPath
}

# Silinen klasörlerin listesini tutmak için bir dizi oluşturur.
$DeletedFolders = @()

# Belirtilen yoldaki tüm alt klasörleri tarar.
Get-ChildItem -Path $Path -Directory | ForEach-Object {
    
    # 3 günden eski klasörleri bulur.
    if ($_.LastWriteTime -lt (Get-Date).AddDays(-$Days)) {
        
        Write-Host "Siliniyor: $($_.FullName) - Son yazılma tarihi: $($_.LastWriteTime.ToString('dd.MM.yyyy HH:mm:ss'))"
        
        # Silinen klasörün bilgilerini bir nesne olarak listeye ekler.
        $DeletedFolders += [PSCustomObject]@{
            KlasörYolu        = $_.FullName
            SilinmeTarihi     = (Get-Date).ToString('dd.MM.yyyy HH:mm:ss')
            SonYazılmaTarihi  = $_.LastWriteTime.ToString('dd.MM.yyyy HH:mm:ss')
        }
        
        # Klasörü ve içindeki her şeyi siler.
        Remove-Item -Path $_.FullName -Recurse -Force -Confirm:$false
    }
}

# HTML raporunu hazırlar.
$HTMLReport = @()

if ($DeletedFolders.Count -gt 0) {
    # Silinen klasörler varsa, HTML tablosu oluşturur.
    $HTMLReport = $DeletedFolders | ConvertTo-Html -Fragment -PreContent "<h3>Silinen Klasörler:</h3>"
    
    # HTML stilini ekler.
    $HTMLReport = @"
<style>
    body { font-family: Arial, sans-serif; }
    h2, h3 { color: #333; }
    table { border-collapse: collapse; width: 100%; }
    th, td { text-align: left; padding: 8px; border: 1px solid #ddd; }
    tr:nth-child(even) { background-color: #f2f2f2; }
    th { background-color: #4CAF50; color: white; }
</style>
<h2>Klasör Temizleme Raporu</h2>
Tarih: $((Get-Date).ToString('dd.MM.yyyy HH:mm:ss'))
$($HTMLReport)
"@
} else {
    # Silinen klasör yoksa, basit bir HTML mesajı hazırlar.
    $HTMLReport = @"
<style>
    body { font-family: Arial, sans-serif; }
    h2, h3 { color: #333; }
</style>
<h2>Klasör Temizleme Raporu</h2>
Tarih: $((Get-Date).ToString('dd.MM.yyyy HH:mm:ss'))
<p>3 günden eski klasör bulunamadı.</p>
"@
}

# HTML raporunu dosyaya yazar.
# Dosya adını 'Rapor_GG.AA.YYYY_SS-DD.html' formatında oluşturur.
$CurrentDate = Get-Date
$ReportFileName = "Rapor_$($CurrentDate.ToString('dd.MM.yyyy_HH-mm')).html"
$ReportFilePath = Join-Path -Path $ReportPath -ChildPath $ReportFileName

$HTMLReport | Out-File -FilePath $ReportFilePath -Encoding utf8

Write-Host "Rapor oluşturuldu: $ReportFilePath"
