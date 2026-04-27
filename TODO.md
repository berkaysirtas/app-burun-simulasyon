# TODO

## Arkaplan Kaldırma — Gerçek Çözüm

Mevcut "Arkaplanı Kaldır" / "Zemin Beyaz" butonları geçici olarak kaldırıldı.
Edge-based flood-fill yöntemi yalnızca düz/uniform zeminlerde çalışıyor;
gerçek hasta fotoğraflarında karmaşık arka plan + saç kenarı için yetersiz.

### Tarayıcı içi çözüm seçenekleri (API ücreti yok)

1. **@imgly/background-removal** — önerilen
   - ONNX Runtime Web + U²-Net (rembg modeli)
   - ~30 MB ilk indirme, sonra cache
   - Tek satır API: `await imglyRemoveBackground(blob)`
   - Photoshop kalitesinde sonuç, her sahnede çalışır
   - PWA cache'e eklenebilir, offline çalışır

2. **MediaPipe Selfie Segmentation**
   - ~3 MB model, WASM
   - Portrelerde çok iyi, saç dahil
   - Sadece insan/portre sahneleri için tasarlanmış
   - En hafif çözüm

3. **ONNX U²-Net manuel entegrasyon**
   - ~80 MB model
   - Tam kontrol, en yüksek kalite
   - Daha fazla glue kod gerekir

### Uygulama planı (seçilen lib ile)

- [ ] CDN ya da bundle ile lib entegrasyonu
- [ ] Header'a tekrar "Arkaplanı Kaldır" butonu
- [ ] Loading overlay UI (CSS + spinner)
- [ ] Sonuç → `warpSource` olarak set + `dispX`/`dispY` reset (mevcut mantık hazır)
- [ ] PWA `sw.js` cache'ine modeli ekle (offline çalışsın)
- [ ] "Zemin Beyaz" butonu opsiyonel — şeffaf alanı beyaz dolduran kısa fonksiyon

### Notlar

- Warp tarafında `warpSource` mimarisi zaten hazır: BG kaldırma sonucu yeni
  warpSource olur, displacement field sıfırlanır. Sonraki warp'lar tek-bicubic
  örnek kalır, blur birikmez.
- Eski flood-fill kodu git history'de mevcut (commit acf6842 öncesi).
