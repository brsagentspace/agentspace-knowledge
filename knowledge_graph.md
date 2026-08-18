# Proje Bilgi Grafiği (Local Knowledge Graph)

Bu doküman, AgentSpace projemiz için yaptığımız tüm araştırmaları, kararları ve mimari bileşenleri birbirine bağlayan yerel (local) bilgi grafiğimizdir. Tıpkı Google Cloud Knowledge Catalog'da olduğu gibi, bu grafiği projenin hafızası olarak kullanacağız. Yeni araştırmalar yaptıkça bu grafiği güncelleyeceğim.

## 🧠 Görsel İlişki Ağı (Context Graph)

Aşağıdaki grafikte şu ana kadar aldığımız kararların ve araştırmaların birbiriyle olan ilişkisini görebilirsiniz:

```mermaid
graph TD
    %% Ana Düğümler (Nodes)
    Project(("AgentSpace Projesi"))
    Org["GitHub Organizasyonu"]
    Arch["Mimari & Altyapı"]
    Assets["Görsel Materyaller (Assets)"]
    
    %% İlişkiler (Edges)
    Project -->|Barındırılır| Org
    Project -->|İnşa edilir| Arch
    Project -->|Kullanır| Assets
    
    %% Organizasyon Detayları
    Org --> BRS["brsagentspace"]
    
    %% Mimari Detayları
    Arch --> Framework["Cross-Platform Çatı"]
    Arch --> Engine["Animasyon Motoru"]
    Arch --> UI["Arayüz & Terminal"]
    
    Framework --> Tauri["Tauri (Mac/Win/Linux için)"]
    Engine --> Phaser["Phaser.js (2D Ofis için)"]
    UI --> Xterm["xterm.js (Terminal Entegrasyonu)"]
    UI --> React["React.js"]
    
    %% Asset Detayları (v2 — 2026-08-18: LimeZu satın alındı ve entegre edildi)
    Assets --> Style["Tarz: Pixel Art 16px (LimeZu)"]
    Assets --> Env["Ofis Çevresi"]
    Assets --> Chars["Ajan Karakterleri"]
    Assets --> Emotes["Durum Emote Balonları"]

    Style --> Env
    Style --> Chars

    Env --> LimeZuOffice["✅ LimeZu Modern Office Revamped (2,50$)"]
    Env --> LimeZuInt["✅ LimeZu Modern Interiors (1,50$)"]
    Chars --> LimeZuChars["✅ LimeZu Premade Characters (18 adet, 4 yön + oturma)"]
    Emotes --> LimeZuEmotes["✅ LimeZu UI Thinking Emotes"]

    LimeZuInt --> License["⚠️ Lisans: dağıtım yasak → git'e girmez (.gitignore)"]
    LimeZuOffice --> License

    %% Reddedilen adaylar
    Assets --> Rejected["❌ Reddedilenler"]
    Rejected --> KenneyRobots["Kenney Robot Pack — yandan görünüm, top-down'a uymadı"]
    Rejected --> KenneyUrban["Kenney RPG Urban — dış mekan seti, ofis içi mobilya yok"]
    Rejected --> LPC["LPC karakterler — ortaçağ teması + CC-BY-SA lisansı"]

    %% Stil Atamaları
    classDef core fill:#a855f7,stroke:#fff,stroke-width:2px,color:#fff;
    classDef arch fill:#3b82f6,stroke:#fff,stroke-width:2px,color:#fff;
    classDef asset fill:#10b981,stroke:#fff,stroke-width:2px,color:#fff;
    classDef org fill:#f59e0b,stroke:#fff,stroke-width:2px,color:#fff;

    class Project core;
    class Arch,Framework,Engine,UI,Tauri,Phaser,Xterm,React arch;
    class Assets,Style,Env,Chars,Emotes,LimeZuOffice,LimeZuInt,LimeZuChars,LimeZuEmotes asset;
    class Org,BRS org;
```

## 📋 Varlık Kataloğu (Entity Catalog)

Grafikteki her bir düğümün (node) detaylı teknik bağlamı:

### 1. Mimari (Architecture)
- **Tauri:** Uygulamayı Mac, Windows ve Linux'ta tüy gibi hafif çalıştırmak için seçtiğimiz ana motor.
- **Phaser.js:** Ajanların masalara oturması, yürümesi ve ofis içinde etkileşime girmesi için kullanılacak 2D oyun/simülasyon motoru.
- **React + xterm.js:** Çoklu terminal ekranlarını ve kullanıcı arayüzünü oluşturmak için kullanılacak web teknolojileri.

### 2. Görsel Materyaller (Assets) — v2 (2026-08-18, UYGULANDI ✅)
- **Tarz:** 16px pixel art, tek tutarlı aile: **LimeZu Modern Interiors + Modern Office Revamped** (toplam ~4$, itch.io). Muratify referans görünümünün kaynağı bu paketlerdir.
- **Ajanlar:** LimeZu premade **insan** karakterleri (robot kararı iptal — bkz. DECISIONS.md §14). 18 karakter; idle/walk 4 yön, masada oturma, telefon animasyonları. "Ajan Ekle" modalında karakter seçici var.
- **Çevre/Ofis:** Room Builder Office duvar/zemin tile'ları + mobilya sheet'i; 6 odalı tilemap (5 çalışma odası + break room), oda başına 6 masa.
- **UI & İfadeler:** LimeZu UI Thinking Emotes — ⛏️ çalışıyor, 💬 düşünüyor, 💤 boşta, ✨ bitti, ❌ engellendi.
- **⚠️ Lisans:** Yeniden dağıtım yasak; repolar public olduğundan asset'ler `.gitignore`'da — git'e asla girmez. Detay: DECISIONS.md §14.

### 3. Yönetim & Dağıtım
- **brsagentspace (GitHub Org):** Tüm kaynak kodların tutulacağı ve ajanların ileride PR/Issue okuyarak etkileşime gireceği merkezi kod deposu organizasyonu.

---
*Not: Yeni kütüphaneler araştırdıkça veya yeni kurallar belirledikçe bu grafiğe yeni Edge (ilişki) ve Node (düğüm) eklenecektir.*
