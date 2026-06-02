# 🎤 Sunum Materyalleri — Kullanım Kılavuzu (Kişi 2)

Bu klasörde sunum için 2 ana materyal var:

| Dosya | Ne işe yarar |
|-------|--------------|
| **SUNUM_DIAGRAMLAR.md** | 10 adet Mermaid diyagramı (mimari, akış şemaları, splitting, coalescing, vb.) |
| **animasyon.html** | Tarayıcıda çalışan canlı animasyon — heap'in malloc/split/free/coalesce ile değişimini gösterir |

---

## 1️⃣ Mermaid Diyagramlarını Kullanmak

### Görüntülemek
- **VS Code:** `SUNUM_DIAGRAMLAR.md` dosyasını aç → sağ üstten "Open Preview" (Ctrl+Shift+V). (Markdown Preview Mermaid eklentisi gerekebilir.)
- **GitHub:** Dosyayı repo'ya push edersen diyagramlar otomatik çizilir.
- **Online:** <https://mermaid.live> → ```` ```mermaid ```` blokları arasındaki kodu yapıştır.

### Sunuma resim olarak almak (PNG/SVG)
1. <https://mermaid.live> aç
2. Diyagram kodunu yapıştır
3. Sağ üstten **Actions → PNG / SVG** indir
4. PowerPoint / Google Slides'a ekle

---

## 2️⃣ Animasyonu Kullanmak

### Sunumda canlı oynatmak (en kolay)
- `animasyon.html` dosyasına çift tıkla → tarayıcıda açılır.
- **"Otomatik Oynat"** ile kendi kendine ilerler (her 2.6 sn'de bir adım).
- **"Sonraki Adım"** ile sen kontrol edersin (anlatırken ideal).
- Sunum sırasında tarayıcıyı tam ekran yap (F11).

### GIF'e çevirmek 🎞️

**Yöntem A — Ekran kaydı (en pratik):**
1. Windows'ta **Xbox Game Bar**: `Win + G` → kayıt başlat (veya `Win + Alt + R`).
2. Animasyonda "Otomatik Oynat"a bas, bir tam tur kaydet.
3. Çıkan MP4'ü GIF'e çevir: <https://ezgif.com/video-to-gif> sitesine yükle.

**Yöntem B — ShareX (ücretsiz, doğrudan GIF):**
1. [ShareX](https://getsharex.com/) indir.
2. "Capture → Screen recording (GIF)" seç, animasyon alanını çiz.
3. Otomatik oynat → bir tur kaydet → GIF hazır.

**Yöntem C — Tarayıcı eklentisi:**
- Chrome/Edge'de "GIF Screen Recorder" tarzı bir eklenti ile pencere bölgesini kaydet.

> 💡 İpucu: GIF için animasyon penceresini ~920px genişlikte tut, kayıt boyutu küçük kalsın.

---

## 3️⃣ Sunum Akış Önerisi (5 dk)

1. **Slayt 1 — Mimari** → Diyagram 1: "Ben (Kişi 2) şu dosyaları yazdım."
2. **Slayt 2 — Bellek düzeni** → Diyagram 2: Header + Payload + Magic number.
3. **Slayt 3 — CANLI ANİMASYON** → `animasyon.html` oynat, malloc→split→free→coalesce'ı göster.
4. **Slayt 4 — malloc/free akışı** → Diyagram 3 ve 4 (akış şemaları).
5. **Slayt 5 — Stratejiler** → Diyagram 7: first-fit vs best-fit.
6. **Slayt 6 — Güvenlik** → Diyagram 10: double-free, geçersiz pointer yakalama.
7. **Kapanış** → Diyagram 8: "Tüm bunlar birbirini şöyle çağırıyor."

---

## 🎯 Konuşurken vurgulanacak 3 ana mesaj

1. **"my_malloc/my_free bir orkestra şefi"** — kendisi az iş yapar, doğru sırayla diğer fonksiyonları çağırır.
2. **Splitting + Coalescing = fragmentation (israf) yönetimi** — belleğin verimli kullanılmasını sağlar.
3. **Güvenlik katmanları** — magic number, double-free ve geçersiz pointer tespiti ile bellek hatalarını yakalar.
