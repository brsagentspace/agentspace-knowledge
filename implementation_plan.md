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

## Doğrulama ve Sonraki Adım
Eğer bu mimari ve entegrasyon mantığı aklınıza yattıysa, lütfen yukarıdaki soruları yanıtlayın. Onayınızın ardından kod deposunun iskeletini (Tauri + React + Vite) yerel bilgisayarınızda kurarak işe resmi olarak başlayabiliriz.
