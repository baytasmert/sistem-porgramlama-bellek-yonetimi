# Proje Raporu

- **Kişi 1:** Büşra Ay
- **Kişi 3:** Demirhan Çakır

## Özet
Bu proje, bellek yönetimi ve dinamik tahsisat üzerine bir uygulama içerir. Temel amaç, ayrılmış bellek bloklarıyla çalışan bir allocator (ayırıcı) tasarlamak ve güvenli çok iş parçacıklı kullanım ile terminal arayüzü sağlamaktır.

## Giriş
Proje, bellek dağıtımı, serbest bırakma, hata kontrolü ve thread-safe (çok iş parçacıklı) senaryoları ele alan bir C uygulamasıdır. Testler, temel işlevselliği doğrulamak için sağlanmıştır.

## Mimari
- Kaynak dosyaları: `src/allocator_core.c`, `src/allocator_ops.c`, `src/allocator_safety.c`, `src/allocator_threadsafe.c`, `src/allocator_terminal_ui.c`, `src/allocator_debug.c`
- Başlıklar: `include/allocator.h`, `include/allocator_threadsafe.h`, `include/allocator_terminal_ui.h`
- Testler: `tests/test_allocator.c`

## Uygulama
Uygulama, bellek bloklarını yöneten çekirdek modüller, güvenlik kontrolleri, thread-safe sarıcı ve terminal tabanlı bir arayüz içerir. Öne çıkan fonksiyonlar ve dosya eşlemeleri rapor içinde incelenmiştir.

## Testler
Testler `tests/test_allocator.c` altında toplanmıştır. Temel senaryolar: tahsisat, serbest bırakma, taşma/taşma kontrolü ve eşzamanlı erişim testleridir.

## Sonuç
Proje, dinamik bellek yönetimi konusundaki temel kavramları uygulamalı olarak göstermektedir. Thread-safe kullanım ve hata denetimleri ile güvenlik artırılmıştır.

## Dosya Listesi (Önemli)
- `src/allocator_core.c`
- `src/allocator_ops.c`
- `src/allocator_safety.c`
- `src/allocator_threadsafe.c`
- `src/allocator_terminal_ui.c`
- `src/allocator_debug.c`
- `include/allocator.h`
- `include/allocator_threadsafe.h`
- `include/allocator_terminal_ui.h`
- `tests/test_allocator.c`

## Referanslar
- Proje kaynak kodu ve ders notları.
