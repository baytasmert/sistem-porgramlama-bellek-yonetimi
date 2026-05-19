# Custom Memory Allocator - Bellek Yönetimi

Sistem Programlama 2026 - Bellek Yönetimi - Dönem Sonu Projesi

## 📌 Hızlı Başlangıç

```bash
# Projeyi derle
make

# Test programını çalıştır
./test_allocator

# İstatistikleri görmek için
allocator_print_stats();
```

## 📚 Dokümantasyon

Detaylı dokümantasyon `docs/` klasöründe:

| Doküman | İçerik |
|---------|--------|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | Sistem mimarisi, bellek düzeni, veri yapıları |
| [API.md](docs/API.md) | Tüm fonksiyonların detaylı API referansı |
| [IMPLEMENTATION.md](docs/IMPLEMENTATION.md) | Person 2'nin implementasyon detayları |

## 🏗️ Proje Yapısı

```
├── include/
│   └── allocator.h                 # Ana header dosyası
├── src/
│   ├── allocator_core.c            # Çekirdek altyapı (Person 1)
│   │   ├─ allocator_init()
│   │   ├─ Free list yönetimi
│   │   ├─ Block list yönetimi
│   │   └─ OS iletişimi (sbrk)
│   ├── allocator_ops.c             # Tahsis operasyonları (Person 2)
│   │   ├─ my_malloc() + splitting
│   │   ├─ my_free() + coalescing
│   │   ├─ my_calloc()
│   │   ├─ allocator_strategy_first_fit()
│   │   └─ allocator_strategy_best_fit()
│   └── allocator_safety.c          # Hata kontrolü (Person 2)
│       ├─ Double-free tespiti
│       ├─ Invalid pointer tespiti
│       ├─ allocator_report_memory_leaks()
│       └─ allocator_print_stats()
├── test/
│   └── test_allocator.c            # Test programı (Person 3)
├── docs/
│   ├── ARCHITECTURE.md
│   ├── API.md
│   └── IMPLEMENTATION.md
└── Makefile
```

## ✨ Temel Özellikler

### Tahsis Fonksiyonları
- ✅ `my_malloc()` - Bellek tahsisi (splitting ile)
- ✅ `my_free()` - Bellek serbest bırakma (coalescing ile)
- ✅ `my_calloc()` - Sıfırlanmış tahsis

### Optimizasyonlar
- ✅ **Block Splitting** - Internal fragmentation azaltma
- ✅ **Block Coalescing** - External fragmentation azaltma
- ✅ **Yerleştirme Stratejileri** - first-fit (hızlı) ve best-fit (verimli)

### Güvenlik
- ✅ **Double-free tespiti** - Aynı blok iki kez serbest bırakılmayı engeller
- ✅ **Invalid pointer tespiti** - Geçersiz pointerler tespit edilir
- ✅ **Magic numbers** - Bellek bozulması kontrolü
- ✅ **Boundary checks** - Taşma ve sınır kontrolleri

### İstatistikler & Raporlama
- ✅ **Bellek sızıntısı raporu** - Serbest bırakılmamış blokları listeler
- ✅ **Kapsamlı istatistikler** - Fragmentation, kullanım oranı, vb.

## 💻 Kullanım Örneği

```c
#include "allocator.h"
#include <stdio.h>

int main() {
    // Best-fit stratejisi kullan (daha verimli)
    allocator_set_strategy(allocator_strategy_best_fit);
    
    // Bellek tahsis et
    int *arr = (int *)my_malloc(10 * sizeof(int));
    if (arr == NULL) {
        fprintf(stderr, "Tahsis başarısız\n");
        return 1;
    }
    
    // Sıfırlanmış bellek tahsis et
    char *buffer = (char *)my_calloc(256, 1);
    
    // Belleği kullan
    arr[0] = 42;
    buffer[0] = 'A';
    
    // İstatistikleri yazdır
    allocator_print_stats();
    
    // Serbest bırak
    my_free(arr);
    my_free(buffer);
    
    // Sızıntı kontrol et
    allocator_report_memory_leaks();
    
    return 0;
}
```

## 📊 Görev Dağılımı

| Kişi | Sorumluluk | Durum |
|------|------------|-------|
| **Person 1** | Çekirdek allocator | ✅ Tamamlandı |
| **Person 2** | Tahsis stratejileri, free, calloc, hata kontrol | ✅ Tamamlandı |
| **Person 3** | Thread safety (mutex), test, Makefile | ⏳ Yapılacak |

### Person 2 - Tamamlanan Görevler

✅ Yerleştirme stratejileri (first-fit, best-fit)  
✅ Block splitting (blok bölme)  
✅ Block coalescing (blok birleştirme)  
✅ `my_malloc()` implementasyonu (splitting ile)  
✅ `my_free()` implementasyonu (coalescing ile)  
✅ `my_calloc()` implementasyonu  
✅ Double-free tespiti  
✅ Invalid pointer tespiti  
✅ Bellek sızıntısı raporu (`allocator_report_memory_leaks()`)  
✅ İstatistikler (`allocator_print_stats()`)  

## 🔧 Yerleştirme Stratejileri

### First-Fit (Varsayılan)
```c
allocator_set_strategy(allocator_strategy_first_fit);
```
- **Hız**: O(n) ama çoğu zaman erken çıkış
- **Avantaj**: Hızlı tahsis
- **Dezavantaj**: Fragmentation'a yatkın

### Best-Fit
```c
allocator_set_strategy(allocator_strategy_best_fit);
```
- **Hız**: O(n) tam tarama
- **Avantaj**: Internal fragmentation azalır
- **Dezavantaj**: First-fit'ten biraz yavaş

## 🎯 Önemli Özellikler

### Magic Numbers
```
0xC0FFEE01 - Tahsis edilmiş blok
0xC0FFEE02 - Serbest blok
```
Bellek bozulması tespit etmek için kullanılır.

### Alignment
- Tüm payload'lar 16-byte aligned
- Modern CPU cache verimli çalışması

### Fragmentation Kontrolü

**Splitting**: 
```
Tahsis öncesi:  [====== 512 byte ======]
Tahsis sonrası: [== 256 ===][== 256 serbest ==]
```

**Coalescing**:
```
Free öncesi:    [serbest][KULLANILAN][serbest]
Free sonrası:   [======= Birleştirilmiş =======]
```

## ⚠️ Bilinen Sınırlamalar

- **Thread-unsafe** - Mutex eklenene kadar çok-thread ortamda güvensiz (Person 3 yapacak)
- **No Realloc** - `realloc()` uygulanmamıştır
- **Manual cleanup** - Garbage collection yoktur

## 📈 Performans İpuçları

| Durum | Önerilen Strateji |
|-------|-------------------|
| Hızlı tahsis isteniyor | First-Fit |
| Bellek verimliliği önemli | Best-Fit + Coalescing |
| Yoğun malloc/free | First-Fit (daha az tarama) |

## 📝 Test Senaryoları

Person 3 tarafından yazılacak test senaryoları:
- Basic allocation/free
- Block splitting
- Block coalescing
- Double-free detection
- Invalid pointer detection
- Memory leak detection
- Thread safety
- Fragmentation analysis

---

## 📖 Kaynak Kodları

- `allocator_core.c` - Çekirdek (300 satır)
- `allocator_ops.c` - Operasyonlar (300 satır)
- `allocator_safety.c` - Güvenlik (150 satır)
- **Toplam**: ~750 satır üretken kod
