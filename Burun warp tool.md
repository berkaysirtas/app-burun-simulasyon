# Burun Warp Aracı

Tarayıcıda çalışan, sıfır bağımlılıklı bir burun estetik simülasyon aracı. Fotoğrafı yükle, sürükle, indir.

---

## Nasıl Çalışır

Kullanıcı fotoğrafı yükler → canvas üzerine render edilir → sürükleme hareketleri mesh warp algoritmasıyla piksel kaydırması yapar → sonuç PNG olarak indirilebilir.

---

## Kullanılan Teknoloji

| Teknoloji | Açıklama |
|---|---|
| HTML5 Canvas API | Görüntü render ve piksel manipülasyonu |
| `getImageData` / `putImageData` | Ham piksel okuma/yazma |
| Bilinear Interpolation | Kayma sırasında kaliteli piksel hesaplama |
| Mesh Warp (Displacement Map) | Fırça merkezinden uzaklığa göre falloff ile piksel kaydırma |

Harici kütüphane kullanılmamıştır. Tarayıcı API'leri yeterlidir.

---

## Algoritma: Mesh Warp

Her sürükleme hareketinde aşağıdaki adımlar çalışır:

1. Mevcut `ImageData` kopyalanır (undo için)
2. Fırça yarıçapı içindeki her piksel için mesafe hesaplanır
3. Falloff katsayısı: `(1 - dist² / radius²)²`
4. Kaynak koordinat: `srcX = x - dx * strength * falloff`
5. Bilinear interpolation ile dört komşu pikselden renk hesaplanır
6. Sonuç piksele yazılır

```
falloff = (1 - dist² / r²)²
srcX    = x - dx · strength · falloff
srcY    = y - dy · strength · falloff
```

Falloff sıfırdan başlayıp fırça merkezi yaklaştıkça 1'e ulaşır. Kenarlar yumuşak geçişlidir.

---

## Dosya Yapısı

Tek dosya uygulamasıdır. Ayrı CSS veya JS dosyası yoktur.

```
index.html
```

Bölümler:

- `<style>` — UI stilleri
- `<div id="app">` — Upload zone + canvas + kontrol paneli
- `<script>` — Tüm uygulama mantığı

---

## Canvas Boyutlandırma

Yüklenen fotoğraf maksimum `800×700` piksele ölçeklenir, orijinal oran korunur:

```js
const scale = Math.min(MAX_W / w, MAX_H / h, 1);
```

Orijinal fotoğraf değiştirilmez; sıfırlama için `originalImageData` saklanır.

---

## Kontroller

| Kontrol | Açıklama |
|---|---|
| Güç (5–80) | Pikselin ne kadar kayacağı (strength * 0.01) |
| Yarıçap (20–150) | Fırça büyüklüğü — küçük: hassas düzenleme, büyük: geniş alan |
| Geri Al | Son sürükleme öncesi duruma döner (maks 20 adım) |
| Sıfırla | Orijinal fotoğrafa döner (undo stack'e ekler) |
| İndir | Canvas içeriğini PNG olarak kaydeder |

---

## Undo Sistemi

Her `mousedown` olayında mevcut `ImageData` stack'e kopyalanır. Maksimum 20 adım saklanır. Stack dolunca en eski kayıt silinir (FIFO).

```js
function pushUndo() {
  if (undoStack.length >= MAX_UNDO) undoStack.shift();
  undoStack.push(new ImageData(
    new Uint8ClampedArray(currentImageData.data),
    currentImageData.width,
    currentImageData.height
  ));
}
```

---

## Desteklenen Formatlar

Giriş: JPG, PNG, WEBP (tarayıcının desteklediği tüm görüntü formatları)  
Çıkış: PNG (`canvas.toDataURL('image/png')`)

---

## Tarayıcı Uyumluluğu

Canvas API ve `getImageData` tüm modern tarayıcılarda desteklenir.

| Tarayıcı | Durum |
|---|---|
| Chrome 90+ | ✓ |
| Firefox 88+ | ✓ |
| Safari 14+ | ✓ |
| Edge 90+ | ✓ |

Mobil cihazlarda touch event'leri (`touchstart`, `touchmove`, `touchend`) desteklenir.

---

## Geliştirici Notları

- `pointer-events: none` overlay canvas sadece fırça gösterimi içindir, tıklamalar ana canvas'a geçer
- `Uint8ClampedArray` kopyası referans yerine değer kopyası yapar — undo güvenlidir
- `getPixel()` fonksiyonu canvas sınırlarını `clamp` ile korur, sınır piksellerinde hata oluşmaz
- Sürükleme hızını `Math.abs(dx) < 0.5` koşuluyla filtreler — gereksiz yeniden render engellenir