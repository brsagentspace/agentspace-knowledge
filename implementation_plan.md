# AgentSpace Sistem Mimarisi ve Entegrasyon Planı

Tauri tabanlı yerel masaüstü uygulamamızın (AgentSpace), Google Cloud Knowledge Catalog (Bilgi Grafiği) ve GitHub Organizasyonunuz (brsagentspace) ile nasıl entegre çalışacağına dair mimari plan.

## Kullanıcı İncelemesi Gerekenler

> [!IMPORTANT]
> **GitHub Entegrasyonu:** Ajanların GitHub deponuza (repo) erişip kod okuması/yazması veya Issue/PR'ları yönetmesi için bir GitHub Personal Access Token (PAT) veya GitHub App kullanmamız gerekecek. GitHub organizasyonunuzun (brsagentspace) tam olarak hangi süreçlerini ajanlara devretmek istiyorsunuz?

> [!QUESTION]
> **Knowledge Graph Görselleştirmesi:** Bilgi Grafiğini (Knowledge Graph) ajanların sadece arka planda (kendi aralarında veri paylaşmak için) kullanmasını mı istiyorsunuz, yoksa arayüzde ofis/terminal ekranına ek olarak üçüncü bir panelde bu grafiğin (örümcek ağı gibi veri noktalarının) görsel bir haritasını görmek ister misiniz?

## 1. Google Cloud Knowledge Catalog (Context Graph) Entegrasyonu

Knowledge Catalog (eski adıyla Dataplex), ajanlara veriler arasındaki anlamsal ilişkileri ve iş bağlamını (business context) öğretmek için kullanılır. Ajanların standart bir metin belleği yerine "kim neyle ilişkili, hangi kod hangi servise bağlı" gibi ilişkisel bir hafızaya sahip olmasını sağlar.

### Mimari Yaklaşım
- **Merkezi Hafıza (Brain):** Knowledge Catalog, AgentSpace'in "ortak beyni" olarak kurgulanacaktır. Tüm ajanlar (Claude, GPT, Gemini) görevlerine başlamadan önce veya karar verirken güncel bağlamı (context) çekmek için bu grafiği sorgulayacaktır.
- **Veri Modeli (Node ve Edge Yapısı):**
  - *Nodelar (Düğümler):* Projeler, Dosyalar, Ajanlar, Görevler, Teknolojiler.
  - *Edge'ler (İlişkiler):* "Ajan-1, A Görevi üzerinde çalışıyor", "B Dosyası, C kütüphanesini gerektirir".
- **Görselleştirme:** Eğer uygulamanın içinde bu haritayı görmek isterseniz, Tauri + React mimarimizin içine `react-force-graph` gibi bir kütüphane gömerek, Knowledge Catalog'dan gelen verileri 2D/3D interaktif bir ağ (network) haritası olarak ofis görselleştirmesinin yanında sunabiliriz.

## 2. GitHub Organizasyonu (brsagentspace) Entegrasyonu

GitHub organizasyonunuz projenin hem barındırıldığı ev hem de ajanların aktif olarak müdahale edeceği bir çalışma alanı olacaktır.

### A. Kod Barındırma ve Dağıtım (CI/CD)
- Uygulamanın tüm kaynak kodları (Tauri, Rust, React, Phaser.js) `brsagentspace` altındaki ana repoda tutulacaktır.
- **Tauri + GitHub Actions:** Tauri, cross-platform derleme için GitHub Actions ile mükemmel bir uyuma sahiptir. Kodunuzu push'ladığınız anda, GitHub sunucularında otomatik olarak Mac (.dmg), Windows (.exe) ve Linux (.AppImage) için derleme yapılacak ve "Release" sekmesine yüklenecektir.

### B. Ajanlar İçin GitHub Yetenekleri (Skills/Tools)
AgentSpace'teki ajanlarınıza doğrudan organizasyonunuzla etkileşim kurma yeteneği (Tools) entegre edeceğiz:
- **Ajan 1 (Code Reviewer):** Yeni bir Pull Request açıldığında kodu okuyup Knowledge Graph'teki kurallara göre otomatik inceleme/yorum yapar.
- **Ajan 2 (Developer):** Siz terminalden "Şu projedeki bug'ı çöz" dediğinizde, ajan Issue'yu okur, kodu kendi terminalinde analiz eder ve çözümü içeren yeni bir PR'ı doğrudan `brsagentspace` repolarına açar.
- Ajanların tüm bu faaliyetleri, uygulamanın ofis (Phaser.js) ekranında animasyonlarla (masa başında çalışma, düşünme vs.) görselleştirilir.

## 3. Temel Faz (Phase 1 / MVP) Özellikleri

Dışarıdan bir ajans (20-30 kişilik bir ekip gücü) gibi çalışacak bu sistemin ilk sürümünde (MVP) sağlam bir temel atmak için şu özellikleri kurguladım:

### A. Çoklu Ajan Orkestrasyonu (Multi-Agent Engine)
- **Eşzamanlı Görev İşleme:** Ajanların (robotların) asenkron olarak farklı iş parçacıklarında (thread/worker) çalışabilmesi. Örneğin; "Ajan 1" frontend kodunu yazarken, "Ajan 2"in eşzamanlı olarak backend API testlerini yazması.
- **Ortak Hafıza Havuzu:** Tüm ajanların Google Cloud Knowledge Catalog (Bilgi Grafiği) üzerinden birbirlerinin state'ini (ne aşamada olduklarını) anlık okuyabilmesi.

### B. Phaser.js 2D Ofis (Canlı Simülasyon)
- **Dinamik State Yansıması:** Ajanın arka plandaki durumu (Idle, Fetching, Processing, Writing) doğrudan 2D Canvas üzerindeki robot animasyonuna (Gezinme, Düşünme, Klavye Yazma) yansıyacak.
- **A-Star Pathfinding:** Yeni görev alan robotun ofis içinde boş bir server/çalışma masası bulup engellere çarpmadan oraya yürümesi.

### C. Gelişmiş Çoklu Terminal (xterm.js Multiplexer)
- **Split-Pane (Bölünmüş) Mimari:** Tmux benzeri bir yapıyla, sağ panelde istenildiği kadar terminalin yatay/dikey bölünebilmesi.
- **Ajan/Log İzolasyonu:** Her ajanın kendi stdout/stderr akışının ayrı bir terminal sekmesine (veya grid'ine) pipe edilebilmesi.

### D. Proje Şablonları ve Kural Motoru (Blueprint Engine)
- **Metodolojik Şablonlar (Methodology Presets):** Yeni bir iş geldiğinde "Bu bir Mobil projesidir ve kod/mimari süreci X projesi gibidir" diyebileceğiniz hazır şablon yapısı. Bu, renkleri veya UI'yi 1-1 kopyalamak (klonlamak) anlamına **gelmez**. Ajanlar, o projedeki temiz kod mimarisini (Clean Architecture), bileşen (component) yapısını ve klasörleme mantığını (süreci) referans alır.
- **Tasarım Süreci Dayatması (Design Process Consistency):** Ajanlar görselleri veya renk paletlerini birebir kopyalamak yerine, sizin belirlediğiniz profesyonel tasarım *standartlarını* (örneğin "Atomic Design prensiplerini kullan", "Boşluk (whitespace) hiyerarşisine dikkat et", "Accessibility kurallarına uy") yeni projenin özgün dinamiklerine uyarlar. Bu sayede her proje görsel olarak benzersiz (unique) olur ama kalite/süreç olarak %100 sizin standart imzanızı (tutarlılığı) taşır.
- **Pazarlama (Marketing) Entegrasyonu İçin Altyapı:** İleride kod bitince devreye girecek "Pazarlama Ajanları" için projenin tamamlanmış bağlamını (ürün özellikleri, marka dili) okuyabilecekleri genişletilebilir veri modeli.

## 4. UI (Kullanıcı Arayüzü) Yerleşim Planı (Wireframe)

Uygulamanın (Tauri penceresi) ana ekran yerleşimi, karmaşıklığı önleyecek ancak profesyonel bir IDE hissiyatı verecek şekilde şöyle tasarlanmıştır:

```text
+-----------------------------------------------------------------------+
|  [Proje: X Müşterisi] | [Şablon: Y Mobil Projesi] | [Ayarlar]         |
+-----------------------------------+-----------------------------------+
|                                   |  [Kurallar/Referanslar (Panel)]   |
|                                   |  > UI Guideline: Minimalist       |
|                                   +-----------------------------------+
|   2D Ofis Simülasyonu             |  [Terminal 1 - Ajan: Backend]     |
|   (Phaser.js WebGL Canvas)        |  > Rust servisi derleniyor...     |
|                                   |                                   |
|   - Çalışan Robot Animasyonları   +-----------------------------------+
|   - Dinamik Masa/Server Objeleri  |                                   |
|   - Kenney.nl Emote Baloncukları  |  [Ana Terminal (User Prompt)]     |
|                                   |  $ Tüm ajanlara: Testleri başlat_ |
+-----------------------------------+-----------------------------------+
|  [Aktif Ajanlar: 2/5] | [CPU/RAM Tüketimi] | [GitHub API Limit: %98]  |
+-----------------------------------------------------------------------+
```

### Bölüm İşlevleri
1. **Sol Panel (%60 - Oyun Motoru):** Görsel geri bildirim alanı. Ajanların hangi masada çalıştığı veya boşta beklediği canlı izlenir.
2. **Sağ Panel Üst (Kurallar/Bağlam):** Projeye özel önceden yüklediğiniz tasarım referanslarının, şablonların ve "Kesin Kurallar" listesinin durduğu (ve ajanların okuduğu) sabit referans paneli.
3. **Sağ Panel Alt (IDE Terminal):** Gerçek işin aktığı, kodların ve logların döküldüğü çoklu xterm.js alanı.
4. **Alt Bar (Status Bar):** Sistem metrikleri (CPU/RAM) ve GitHub/Cloud API rate limitlerinin takibi.

## Doğrulama ve Geri Bildirim
Deneyimli bir Full-Stack Engineer olarak, 20-30 kişilik bir ekibin gücünü tek bir ekrandan yönetmek için bu yerleşim ve temel özellikler sizce uygun mu? Özellikle terminal yapısında veya 2D ofis etkileşiminde değiştirmek/eklemek istediğiniz bir temel fonksiyon var mı?
