# Kişi 2 — Sunum Diyagramları (Mermaid)

> Bu dosyadaki diyagramlar **GitHub**, **VS Code (Markdown Preview)** ve <https://mermaid.live> üzerinde otomatik çizilir.
> Sunumda kullanmak için: diyagrama sağ tık → "Resmi farklı kaydet" veya mermaid.live'da PNG/SVG export.

---

## 1. Bellek Düzeni — Bir Blok Nasıl Görünür?

```mermaid
flowchart LR
    subgraph BLOCK["BİR BELLEK BLOĞU"]
        direction LR
        H["📋 HEADER (künye)<br/>size<br/>is_free<br/>magic<br/>next/prev_free<br/>next/prev_all"]
        P["📦 PAYLOAD<br/>(kullanıcının verisi)<br/><br/>my_malloc bu adresi döndürür ➡️"]
    end

    H -->|"block + 1"| P
    P -.->|"ptr - 1"| H

    style H fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style P fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
```

> 🔑 **Magic Number:** `0xC0FFEE01` = DOLU blok · `0xC0FFEE02` = BOŞ blok
> Künye bozulursa magic yanlış olur → allocator hemen yakalar.

---

## 2. `my_malloc()` Akış Şeması

```mermaid
flowchart TD
    START([my_malloc çağrıldı]) --> Z{size == 0?}
    Z -->|Evet| RETNULL1[NULL döndür]
    Z -->|Hayır| INIT{Allocator<br/>kuruldu mu?}
    INIT -->|Hayır| DOINIT["allocator_init()<br/>OS'ten 64KB iste 🔵K1"]
    INIT -->|Evet| ALIGN
    DOINIT --> ALIGN["allocator_align_size()<br/>boyutu 16'nın katına yuvarla 🔵K1"]
    ALIGN --> STRAT["Strateji ile blok bul<br/>first_fit / best_fit ⭐K2"]
    STRAT --> FOUND{Boş blok<br/>bulundu mu?}

    FOUND -->|Evet| SPLIT["split_block()<br/>gerekirse böl ⭐K2"]
    SPLIT --> REMOVE["free list'ten çıkar 🔵K1"]
    REMOVE --> MARK["is_free=0<br/>magic=MAGIC_ALLOC"]
    MARK --> STATS1["istatistik +boyut"]
    STATS1 --> RET1([payload adresi döndür ✅])

    FOUND -->|Hayır| OS["request_from_os()<br/>OS'ten yeni yer iste 🔵K1"]
    OS --> OSCHK{Başarılı mı?}
    OSCHK -->|Hayır| RETNULL2[NULL döndür ❌]
    OSCHK -->|Evet| MARK

    style START fill:#fff3cd,stroke:#ff9800,stroke-width:2px
    style RET1 fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style STRAT fill:#ffe0b2
    style SPLIT fill:#ffe0b2
```

---

## 3. `my_free()` Akış Şeması — Güvenlik Kapıları

```mermaid
flowchart TD
    START([my_free ptr çağrıldı]) --> NULLCHK{ptr == NULL?}
    NULLCHK -->|Evet| DONE1([Hiçbir şey yapma, çık])
    NULLCHK -->|Hayır| TOBLOCK["payload_to_block ptr<br/>künyeye dön + magic kontrol 🔵K1"]

    TOBLOCK --> VALID{Magic<br/>geçerli mi?}
    VALID -->|Hayır| ERR1["🚨 HATA: geçersiz pointer<br/>çık"]
    VALID -->|Evet| DBLCHK{Blok zaten<br/>boş mu?}

    DBLCHK -->|Evet| ERR2["🚨 HATA: DOUBLE-FREE<br/>çık"]
    DBLCHK -->|Hayır| STATS["istatistik -boyut<br/>aktif blok -1"]

    STATS --> ADDFREE["add_to_free_list<br/>boş listeye ekle 🔵K1"]
    ADDFREE --> COAL["coalesce_blocks<br/>komşularla birleştir ⭐K2"]
    COAL --> DONE2([Bitti ✅])

    style START fill:#fff3cd,stroke:#ff9800,stroke-width:2px
    style ERR1 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ERR2 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style DONE2 fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style COAL fill:#ffe0b2
```

---

## 4. Block Splitting (Bölme) — Önce / Sonra

```mermaid
flowchart TB
    subgraph ONCE["ÖNCE: my_malloc 112 byte ister"]
        B1["📋H | 🟩 512 byte BOŞ blok"]
    end

    ONCE --> ARROW["split_block böler ⭐K2"]

    subgraph SONRA["SONRA: 112 verildi, 400 boşta kaldı"]
        B2["📋H | 🟦 112 DOLU"]
        B3["📋H | 🟩 ~400 BOŞ → free list'e döner"]
        B2 --- B3
    end

    ARROW --> SONRA

    style B1 fill:#c8e6c9,stroke:#2e7d32
    style B2 fill:#bbdefb,stroke:#1565c0,stroke-width:2px
    style B3 fill:#c8e6c9,stroke:#2e7d32
    style ARROW fill:#ffe0b2,stroke:#e65100
```

> 💡 Amaç: **internal fragmentation** (iç israf) azaltmak. Kalan parça çok küçükse bölme yapılmaz.

---

## 5. Block Coalescing (Birleştirme) — 2 Adım

```mermaid
flowchart TB
    subgraph S0["BAŞLANGIÇ: ortadaki blok free edildi"]
        A0["🟩 A BOŞ"] --- M0["🟦 B DOLU→boşaltıldı"] --- C0["🟩 C BOŞ"]
    end

    S0 --> STEP1["ADIM 1: soldaki A boş mu? → A, B'yi yutar ⭐K2"]

    subgraph S1["ADIM 1 SONRASI"]
        AB["🟩 A+B birleşti"] --- C1["🟩 C BOŞ"]
    end
    STEP1 --> S1

    S1 --> STEP2["ADIM 2: sağdaki C boş mu? → A+B, C'yi de yutar ⭐K2"]

    subgraph S2["SONUÇ: tek büyük boş blok"]
        ABC["🟩🟩🟩 A+B+C = TEK BÜYÜK BOŞ BLOK"]
    end
    STEP2 --> S2

    style M0 fill:#bbdefb,stroke:#1565c0
    style ABC fill:#a5d6a7,stroke:#1b5e20,stroke-width:3px
```

> 💡 Amaç: **external fragmentation** (dış israf) azaltmak. Neden 2 adım? Sol birleşince merkez blok değişir, sağ kontrol yeni merkez üzerinden yapılır.

---

## 6. First-Fit vs Best-Fit Karşılaştırma

```mermaid
flowchart TB
    REQ["İstek: 100 byte lazım"]
    REQ --> FF & BF

    subgraph FF["🏃 FIRST-FIT (İlk Uyan)"]
        direction TB
        F1["🟩 120 ← İLK uygun, HEMEN AL"]
        F2["🟩 105"]
        F3["🟩 500"]
        F1 -.-> F2 -.-> F3
    end

    subgraph BF["🎯 BEST-FIT (En İyi Uyan)"]
        direction TB
        G1["🟩 120"]
        G2["🟩 105 ← EN YAKIN, bunu seç"]
        G3["🟩 500"]
        G1 -.-> G2 -.-> G3
    end

    FF --> FFR["✅ Hızlı (erken çıkar)<br/>❌ İsraf olabilir"]
    BF --> BFR["✅ Az israf<br/>❌ Tümünü gezer, yavaş"]

    style F1 fill:#fff59d,stroke:#f9a825,stroke-width:3px
    style G2 fill:#a5d6a7,stroke:#1b5e20,stroke-width:3px
```

---

## 7. Fonksiyon Çağrı Grafiği — Kim Kimi Çağırır?

```mermaid
flowchart LR
    USER([👤 KULLANICI])

    USER --> MALLOC["my_malloc ⭐"]
    USER --> FREE["my_free ⭐"]
    USER --> CALLOC["my_calloc ⭐"]

    CALLOC --> MALLOC
    CALLOC --> MEMSET["memset (sıfırla)"]

    MALLOC --> ALIGN2["align_size 🔵"]
    MALLOC --> INIT2["init 🔵"]
    MALLOC --> STRAT2["first/best_fit ⭐"]
    MALLOC --> SPLIT2["split_block ⭐"]
    MALLOC --> REM2["remove_free_list 🔵"]
    MALLOC --> OS2["request_from_os 🔵"]

    FREE --> P2B["payload_to_block 🔵"]
    FREE --> ADD2["add_to_free_list 🔵"]
    FREE --> COAL2["coalesce_blocks ⭐"]

    style MALLOC fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style FREE fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style CALLOC fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style STRAT2 fill:#fff3cd
    style SPLIT2 fill:#fff3cd
    style COAL2 fill:#fff3cd
```

> ⭐ = Kişi 2 yazdı · 🔵 = Kişi 1 yazdı

---

## 8. Bir Bloğun Yaşam Döngüsü (State Diagram)

```mermaid
stateDiagram-v2
    [*] --> BOS: allocator_init / split sonrası
    BOS --> DOLU: my_malloc seçer<br/>(magic=ALLOC, is_free=0)
    DOLU --> BOS: my_free<br/>(magic=FREE, is_free=1)
    BOS --> BIRLESMIS: coalesce_blocks<br/>komşu boşsa
    BIRLESMIS --> DOLU: tekrar my_malloc
    DOLU --> HATA: my_free iki kez!
    HATA --> DOLU: double-free reddedildi

    note right of BOS: free list içinde bekler
    note right of DOLU: kullanıcıda
    note right of HATA: 🚨 güvenlik yakalar
```

---

## 9. Güvenlik Katmanları (Kişi 2)

```mermaid
flowchart TD
    PTR["Gelen pointer / blok"] --> L1

    L1{"1️⃣ Magic doğru mu?<br/>C0FFEE01/02"}
    L1 -->|Hayır| X1["🚨 Bellek bozulması<br/>veya sahte pointer"]
    L1 -->|Evet| L2

    L2{"2️⃣ Boyut > 0 mı?"}
    L2 -->|Hayır| X2["🚨 Geçersiz blok"]
    L2 -->|Evet| L3

    L3{"3️⃣ Zaten boş mu?<br/>(free sırasında)"}
    L3 -->|Evet| X3["🚨 DOUBLE-FREE"]
    L3 -->|Hayır| OK["✅ İşleme devam"]

    style X1 fill:#ffcdd2,stroke:#c62828
    style X2 fill:#ffcdd2,stroke:#c62828
    style X3 fill:#ffcdd2,stroke:#c62828
    style OK fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
```
