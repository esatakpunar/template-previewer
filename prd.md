# HTML ve PDF Önizleyici PRD

## Proje Amacı
Bu proje, kullanıcıların HTML template'lerini ve veri kaynaklarını girerek, sonuçları hem HTML hem de PDF formatında önizleyebilecekleri bir web uygulamasıdır.

## Kullanıcı Arayüzü Gereksinimleri

### Ana Sayfa Düzeni
- Sayfa dikey olarak iki ana bölüme ayrılacak:
  1. Sol Panel
  2. Sağ Panel

### Sol Panel Özellikleri
1. HTML Template Giriş Alanı
   - Geniş bir metin editörü
   - Syntax highlighting desteği
   - Template değişkenlerini vurgulama

2. Data Source Giriş Alanı
   - JSON formatında veri girişi
   - Syntax highlighting desteği
   - JSON validasyonu

3. Kontrol Butonları
   - "Run" butonu
   - Template ve veri kaynağını backend'e gönderme

### Sağ Panel Özellikleri
1. Sekmeli Görünüm
   - HTML Önizleme sekmesi
   - PDF Önizleme sekmesi

2. HTML Önizleme
   - Template ve veri kaynağının birleştirilmiş sonucu
   - Responsive görünüm

3. PDF Önizleme
   - PDF dosyasının önizlemesi
   - Zoom in/out özellikleri

## Teknik Gereksinimler
- Frontend Framework: Vue.js
- UI Kütüphanesi: Tailwind CSS
- Code Editor: Monaco Editor
- PDF Görüntüleyici: PDF.js
- Responsive tasarım
- Modern ve kullanıcı dostu arayüz
- Dark/Light mode desteği
- Error notification sistemi
- Detaylı hata yönetimi ve görüntüleme

## Backend Entegrasyonu
- API Endpoint: `/report/pdfTest`
- Request Format:
  ```json
  {
    "template": "HTML template string",
    "dataSource": "JSON data string"
  }
  ```
- Response Format:
  ```json
  {
    "htmlContent": "Rendered HTML string",
    "pdfContent": "PDF byte array"
  }
  ```

## Kullanıcı Deneyimi
- Yükleme durumları için loading göstergeleri
- Hata mesajları için toast bildirimleri
- Kullanıcı dostu hata yönetimi
- Responsive tasarım ile mobil uyumluluk
- Dark/Light mode geçiş özelliği
- Detaylı hata mesajları ve bildirimler

## Güvenlik Gereksinimleri
- XSS koruması
- Input validasyonu
- Rate limiting
- CORS politikaları
- Base64 to blob dönüşüm güvenliği 