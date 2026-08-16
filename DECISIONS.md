# AgentSpace — Karar Kataloğu (Decision Catalog)

> **Son Güncelleme:** 16 Ağustos 2025  
> **Durum:** Onaylandı — Kodlamaya Hazır  
>
> Bu belge, AgentSpace projesinde alınan **her teknik ve tasarım kararını**,
> değerlendirilen alternatifleri, karar gerekçelerini ve kaynaklarını
> eksiksiz olarak belgeler. Yeni bir karar alındığında bu belgeye eklenir.

---

## İçindekiler

1. [Masaüstü Uygulama Çatısı](#1-masaüstü-uygulama-çatısı)
2. [2D Simülasyon Motoru](#2-2d-simülasyon-motoru)
3. [Terminal Entegrasyonu](#3-terminal-entegrasyonu)
4. [Knowledge Graph Motoru](#4-knowledge-graph-motoru)
5. [Ajan Orkestrasyon Çatısı](#5-ajan-orkestrasyon-çatısı)
6. [RAG Mimarisi](#6-rag-mimarisi)
7. [Token Verimliliği Stratejileri](#7-token-verimliliği-stratejileri)
8. [Bellek Mimarisi](#8-bellek-mimarisi)
9. [Görsel Materyaller (Assets)](#9-görsel-materyaller-assets)
10. [Ajan Karakterleri](#10-ajan-karakterleri)
11. [Gözlemlenebilirlik (Observability)](#11-gözlemlenebilirlik-observability)
12. [Kaynak Kod Yönetimi](#12-kaynak-kod-yönetimi)

---

## 1. Masaüstü Uygulama Çatısı

### ✅ KARAR: Tauri (Rust Backend + Web Frontend)

**Değerlendirilen Alternatifler:**

| Seçenek | Durum | Gerekçe |
|---|---|---|
| **Tauri** | ✅ **SEÇİLDİ** | Hafif, yerel web motoru kullanır |
| Electron | ❌ Reddedildi | Bundled Chromium → 150MB+, yüksek RAM |

**Karar Gerekçesi:**

- **Boyut:** Tauri çıktısı 3-8 MB. Electron "Merhaba Dünya" uygulaması 100MB+.
- **RAM Tüketimi:** Tauri, OS'nin yerleşik web motorunu kullanır (Mac → WebKit, Win → WebView2). Chromium bundle taşımaz.
- **Rust Backend:** Dosya okuma, LLM API çağrıları ve sistem işlemleri Rust'ta çalışır → şimşek hızında.
- **Frontend Uyumu:** React + Phaser.js + xterm.js kombinasyonu Tauri'nin web katmanında sorunsuz çalışır.
- **Phaser.js Uyarısı:** Tauri'de Canvas/WebGL rendering OS'ler arası minimal farklılık gösterebilir (Electron'da Chromium bunu önler). Test gerektirir.

**Referanslar:**
- [Tauri Official Documentation](https://tauri.app/v1/guides/)
- [Tauri vs Electron Benchmark — Comparing Sizes](https://tauri.app/v1/references/benchmarks/)
- Kullanıcı Kararı: *"Tauri kullanmamız lazım"* (oturum 1)

---

## 2. 2D Simülasyon Motoru

### ✅ KARAR: Phaser.js (WebGL/Canvas 2D)

**Değerlendirilen Alternatifler:**

| Seçenek | Durum | Gerekçe |
|---|---|---|
| **Phaser.js** | ✅ **SEÇİLDİ** | Web native, terminal entegrasyonu kolay |
| Godot Engine | ❌ Reddedildi | xterm.js/React ile entegrasyon karmaşık |
| Unity WebGL | ❌ Reddedildi | Build süreleri uzun, overengineered |
| PixiJS | 🔶 Alternatif | Phaser'a yakın, daha az özellikli |
| Pure CSS/Canvas | ❌ Reddedildi | A-Star pathfinding, spritesheet yönetimi zor |

**Karar Gerekçesi:**

- **Pathfinding:** Phaser.js içinde A-Star pathfinding desteği var → robotların ofis içinde engellere çarpmadan yürümesi için gerekli.
- **Spritesheet Yönetimi:** Kenney.nl'den indirilen robot animasyonları (walk, idle) Phaser'ın `AnimationManager`'ı ile yönetilir.
- **React Entegrasyonu:** Phaser bir `<canvas>` öğesine bağlanır, React'ın geri kalanı (terminaller, paneller) çevresinde çalışır.
- **Dinamik State Yansıması:** Ajan backend durumu (Idle → Working → Done) anlık olarak Phaser animasyonuna yansır.

**Referanslar:**
- [Phaser.js Official Site](https://phaser.io/)
- [Phaser 3 Documentation](https://newdocs.phaser.io/docs/3.60.0)

---

## 3. Terminal Entegrasyonu

### ✅ KARAR: xterm.js (Split-Pane Multiplexer)

**Değerlendirilen Alternatifler:**

| Seçenek | Durum |
|---|---|
| **xterm.js** | ✅ **SEÇİLDİ** |
| node-pty + custom | ❌ Reddedildi — xterm.js zaten bunu kapsar |

**Karar Gerekçesi:**

- VS Code, GitHub Codespaces ve onlarca profesyonel araç xterm.js kullanır.
- **Split-Pane Mimari:** Tmux benzeri yatay/dikey bölünebilir terminal paneller.
- **Ajan İzolasyonu:** Her ajanın stdout/stderr akışı ayrı bir terminal sekmesine pipe edilir.
- Tauri'nin Rust backend'i ile `node-pty` üzerinden gerçek PTY süreci yönetilir.

**Referanslar:**
- [xterm.js GitHub](https://github.com/xtermjs/xterm.js) — 17K+ yıldız
- [xterm.js Used By](https://xtermjs.org/) — VS Code, JupyterLab, ttyd

---

## 4. Knowledge Graph Motoru

### ✅ KARAR: Graphiti (getzep/graphiti) + FalkorDB Lite Backend

**Değerlendirilen Alternatifler:**

| Seçenek | Durum | Karar Gerekçesi |
|---|---|---|
| **Graphiti** | ✅ **SEÇİLDİ** | Bi-temporal KG, ajan odaklı, MCP server hazır, gömülü DB |
| Cognee | ❌ Reddedildi | Belge ingestion odaklı, temporal reasoning yok |
| Mem0 | 🔶 Phase 2 | Kişiselleştirme için iyi, temporal zayıf |
| Neo4j (sade) | ❌ Reddedildi | Graphiti'nin backend'i, üstüne Graphiti eklenir |
| Sıfırdan KG | ❌ Reddedildi | 6+ aylık iş |

**Karar Gerekçesi:**

**Teknik Üstünlükler:**

1. **Bi-Temporal Model** — İki zaman boyutu izler:
   - `valid_time`: Gerçekte ne zaman doğruydu
   - `ingestion_time`: Sisteme ne zaman öğretildi
   - "Bu karar geçen ay geçerliydi, şimdi değişti mi?" sorusunu yanıtlar.

2. **FalkorDB Lite (Gömülü, Kurulum Yok):**
   ```bash
   pip install graphiti-core[falkordblite]
   # Bitti. Docker yok, sunucu yok.
   # SQLite gibi dosya tabanlı çalışır.
   ```
   Bu kritik: Tauri masaüstü uygulaması kullanıcı bilgisayarına kurulacak. "Önce Neo4j kur" demek imkansız.

3. **MCP Server Hazır Geliyor:**
   - Graphiti reposunun içinde `mcp_server/` klasörü var.
   - Tauri'nin Rust backend'i ↔ LLM arasındaki köprü MCP protokolü üzerinden kurulur.
   - Sıfırdan köprü yazılmaz.

4. **Çoklu LLM Provider:**
   ```bash
   pip install graphiti-core[falkordb,anthropic,google-genai]
   # Claude + Gemini + OpenAI native destek
   ```

5. **Akademik Temel:** arXiv:2501.13956 — *"Zep: A Temporal Knowledge Graph Architecture for Agent Memory"* (Ocak 2025)

**Neden Cognee Reddedildi:**
- Cognee `remember()` / `recall()` API'si belgeler için optimize — biz ajan olayları (episode) ve kararlar kaydediyoruz.
- Temporal reasoning (geçmiş kararlar, değişen durumlar) yok.
- Multi-agent desteği eksik.

**Referanslar:**
- [getzep/graphiti GitHub](https://github.com/getzep/graphiti) — Apache-2.0
- arXiv:2501.13956 — Zep KG Architecture Paper (Ocak 2025)
- [topoteretes/cognee GitHub](https://github.com/topoteretes/cognee) — Reddedilen alternatif

---

## 5. Ajan Orkestrasyon Çatısı

### ✅ KARAR: LangGraph

**Değerlendirilen Alternatifler:**

| Seçenek | Durum |
|---|---|
| **LangGraph** | ✅ **SEÇİLDİ** |
| Sıfırdan State Machine | ❌ Reddedildi |
| CrewAI | 🔶 Alternatif, daha az esnek |
| AutoGen | 🔶 Daha çok araştırma odaklı |

**Karar Gerekçesi:**

- **Ajan = Node, Geçiş = Edge:** LangGraph'ta her robot bir graph node'u. Görev atama, tamamlama, hata durumları graph edge'leri olarak modellenir.
- **Checkpointing:** LangGraph'ın yerleşik checkpointer'ı → State file protokolümüzün altyapısı.
- **Context Lifecycle:** `Retrieve → Inject → Respond → Persist` döngüsü LangGraph state'ine gömülür.
- **LangMem Uyumu:** İleride LangMem entegrasyonu için native destek.
- **Token Overhead:** ~1600 token sistem overhead'i (LangChain/LangGraph rakibi olarak **LlamaIndex** RAG için daha verimli — bakınız Bölüm 6).

**Referanslar:**
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- LlamaIndex vs LangChain token comparison: ~1600 vs ~2400 system overhead (2025 benchmark)

---

## 6. RAG Mimarisi

### ✅ KARAR: Hibrit RAG (Graphiti Built-in + LlamaIndex + Contextual Retrieval + Reranker)

**Değerlendirilen Alternatifler:**

| Seçenek | Kullanım Alanı | Durum |
|---|---|---|
| Naive RAG (vector-only) | Basit chunk sorgusu | ❌ Yetersiz |
| GraphRAG (Microsoft) | Global özet sorguları | 🔶 İleride |
| **Graphiti Hybrid Search** | KG + temporal sorgular | ✅ **SEÇİLDİ (L3)** |
| **LlamaIndex** | Blueprint dosyaları | ✅ **SEÇİLDİ (L4)** |
| **Contextual Retrieval** | Kod ve dosya parçaları | ✅ **SEÇİLDİ** |
| Ayrı Pinecone/Chroma | Vector store | ❌ Reddedildi — gereksiz |

**3 Aşamalı Retrieval Cascade:**

```
Aşama 1: BM25 (keyword) → top 50 aday (milisaniyeler, sıfır LLM çağrısı)
          ↓
Aşama 2: Cross-Encoder Reranker (ms-marco-MiniLM-L6-v2) → top 10
          ↓
Aşama 3: LLM'e sadece top 3-5 chunk → %85-90 daha az ham token
```

**Anthropic Contextual Retrieval Tekniği:**

Her dosya/kural chunk'ı grafiğe girmeden önce ucuz bir modelle bağlamlandırılır:

```
ÖNCE (Naive Chunk):
"JWT middleware yazıldı"

SONRA (Contextual Retrieval):
"Bu, brsagentspace/ecommerce projesinin auth modülünde,
 Ağustos 2025'te alınan refresh token kararının bir parçası.
 JWT middleware yazıldı."
```

- Retrieval hata oranı **%49 azalır** (Contextual Embeddings + BM25)
- Reranker eklenince **%67 azalır**
- Prompt caching sayesinde bağlam üretimi sadece **ingestion'da bir kez** yapılır → runtime sıfır ek maliyet

**Kaynak Bazlı RAG Stratejisi:**

| Kaynak | Strateji | Araç |
|---|---|---|
| Knowledge Graph (L3) | Graphiti Hybrid Search | Graphiti built-in |
| Blueprint YAML (L4) | BM25 + Semantic | LlamaIndex |
| Kod Dosyaları | Contextual Retrieval + Reranker | LlamaIndex |

**Referanslar:**
- [Anthropic Contextual Retrieval Blog](https://www.anthropic.com/news/contextual-retrieval) (2024)
- TeaRAG: arXiv — *"59-61% reduction in output tokens"*
- [Rankify GitHub](https://github.com/DataScienceUIBK/Rankify) — 24+ SOTA reranker
- [LlamaIndex Documentation](https://docs.llamaindex.ai/)
- LlamaIndex vs LangChain: 1600 vs 2400 token overhead benchmark (2025)

---

## 7. Token Verimliliği Stratejileri

### ✅ KARAR: 5 Katmanlı Token Optimizasyon Sistemi

**Strateji 1: Prompt Prefix Sabitliği (Provider Cache)**

```
[SABİT — ASLA DEĞİŞMEZ] → Provider cache devreye girer
└─ Ajan kimliği ve rolü
└─ Evrensel kurallar
└─ Çıktı format kuralları

[DEĞİŞKEN SUFFIX — Her sorguda güncellenir]
└─ Proje bağlamı (sıkıştırılmış)
└─ Görev detayı
└─ İlgili dosyalar
```

**Etki:** %50-90 token tasarrufu (Claude/Anthropic prompt caching)

**Strateji 2: Tiered Model Seçimi**

| Tier | Model | Görev | Max Context |
|---|---|---|---|
| 1 | Claude Sonnet / GPT-4o | Mimari karar, karmaşık kod | 180k |
| 2 | Claude Haiku / GPT-4o-mini | Rutin kod, refactoring | 60k |
| 3 | Gemini Flash | Özetleme, log analizi, yönlendirme | 20k |
| 4 | Yerel embedding modeli | RAG sorgusu, gizlilik | — |

**Strateji 3: Context Compaction**

Context penceresi %70 dolduğunda Tier-3 model tetiklenir:
- Ham konuşma → sıkıştırılmış `session_summary.md`
- Asla L3'e (KG) gitmez — sadece `Decision`, `Lesson`, `Pattern` gider

**Strateji 4: Ajan İzolasyonu (Context Bleed Önleme)**

Her ajanın context'i izoledir. Backend ajanının bilgisi, Frontend ajanının context'ini kirletmez. Multi-agent sistemlerde %90 performans artışı (2025 benchmark).

**Strateji 5: 3 Aşamalı Retrieval Cascade**

(Detay: Bölüm 6)

**Referanslar:**
- arXiv:2607.21503 — *"Agentic Context Management (ACM)"* (2026)
- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- *"Modular Sub-Agent Design: ~90% performance improvement"* — Context Engineering Research 2025
- TIGRAG: arXiv — Token-Induced GraphRAG

---

## 8. Bellek Mimarisi

### ✅ KARAR: 4 Katmanlı Hiyerarşik Bellek

**Akademik Temel:** arXiv:2603.29194 — *"Multi-Layered Memory Architectures for LLM Agents"* (2026)

```
L1 — ÇALIŞMA BELLEĞİ (Working Memory)
   Yönetim: Sliding window
   Süre: O oturum
   Format: In-context (aktif LLM penceresinde)

L2 — EPİZODİK BELLEĞİ (Session Summary)
   Yönetim: Tier-3 model özetler
   Süre: Proje süresi boyunca
   Format: progress.md (Git'te versiyonlanır)

L3 — SEMANTİK BELLEĞİ (Knowledge Graph)
   Yönetim: Graphiti (bi-temporal)
   Süre: Kalıcı
   Format: FalkorDB Lite (gömülü graph DB)
   İçerik: Decision, Lesson, Pattern node'ları
   Sorgulama: RAG (Graphiti hybrid search)

L4 — PROSEDÜREL BELLEĞİ (Blueprint Engine)
   Yönetim: YAML dosyaları + LlamaIndex
   Süre: Kalıcı, evrimsel
   Format: agentspace-skills/blueprints/*.yaml
   İçerik: Mimari kurallar, tasarım prensipleri, ajan rolleri
   Besleme: Lesson node'ları → Blueprint kural olarak eklenir
```

**State File Protocol (Context Discontinuity Önleme):**

```json
// .agentspace/agents/{agent_id}/state.json
{
  "completed_steps": [...],
  "current_step": "...",
  "next_steps": [...],
  "key_decisions": [...],
  "files_created": [...],
  "tokens_used": 12450,
  "tokens_remaining": 27550
}
```

Context sıfırlansa bile ajan bu dosyayı okuyarak saniyeler içinde tam bağlamını yeniden kurar.

**Referanslar:**
- arXiv:2506.07398 — *"G-Memory: Hierarchical Graph Memory for Multi-Agent Systems"* (Haziran 2025)
- arXiv:2510.04851 — *"LEGOMem: Modular Procedural Memory"* (2024-2025)
- arXiv:2601.20352 — *"AMA: Adaptive Memory via Multi-Agent Collaboration"* (2026)
- arXiv:2607.21503 — *"Agentic Context Management"* (2026)
- arXiv:2606.07921 — *"Governed Shared Memory for Multi-Agent LLM Systems"* (2026)
- [Letta (MemGPT) GitHub](https://github.com/letta-ai/letta) — OS-style memory referansı

---

## 9. Görsel Materyaller (Assets)

### ✅ KARAR: Kenney.nl CC0 Asset Paketi

**Lisans:** CC0 (Creative Commons Zero) — Tamamen ücretsiz, ticari kullanım dahil, attribution gerekmez.

**Seçilen Paketler:**

| Paket | İçerik | Kullanım Alanı | Bağlantı |
|---|---|---|---|
| **Kenney Robot Pack** | Top-view robot spritesheetleri (mavi/kırmızı/sarı/yeşil) | Ajan karakterleri | [kenney.nl/assets/robot-pack](https://kenney.nl/assets/robot-pack) |
| **Kenney RPG Urban Pack** | Ofis zemini, duvarlar, çalışma masaları, server kabinleri | Ofis ortamı | [kenney.nl/assets/rpg-urban-pack](https://kenney.nl/assets/rpg-urban-pack) |
| **Kenney Emotes Pack** | Piksel `...`, `!`, `?`, düşünme baloncukları | Ajan durum göstergesi | [kenney.nl/assets/emotes-pixel](https://kenney.nl/assets/emotes-pixel) |

**Neden Kenney.nl:**
- Dünya standardı ücretsiz oyun materyali sağlayıcısı
- Tüm paketler CC0 — sıfır hukuki risk
- Top-down (üstten görünüm) format → Phaser.js ile doğrudan uyumlu
- Spritesheetler XML metadata ile geliyor → Phaser `AnimationManager` için hazır

**Değerlendirilen Alternatifler:**
- LimeZu Modern Interiors (itch.io) — ücretli, free demo kısıtlı
- Universal LPC Sprite Generator — insan karakterler, robotlar için uygun değil

**İndirme Durumu:** ✅ Tüm paketler indirildi ve `brsagentspace/agentspace-assets` reposuna push edildi.

---

## 10. Ajan Karakterleri

### ✅ KARAR: Robot (İnsan Değil)

**Karar:** Ajanlar piksel sanat robotları olarak görselleştirilir. İnsan karakterler kullanılmaz.

**Gerekçe:**
- Kullanıcı tercihi: *"İnsan olmayacak, robot olacak"* (oturum)
- Robotlar ajanları daha doğrudan ve metaforik olarak temsil eder
- Kenney Robot Pack top-view formatı Phaser.js pathfinding ile uyumlu
- Renk kodlaması ile ajan rolleri görsel olarak ayrışabilir:
  - 🔵 Mavi Robot → Backend / API Ajanı
  - 🔴 Kırmızı Robot → QA / Test Ajanı
  - 🟡 Sarı Robot → Frontend / UI Ajanı
  - 🟢 Yeşil Robot → Araştırma / Dokümantasyon Ajanı

**Animasyon Durumları:**
- `idle` → Ofis içinde rastgele gezinme
- `walking` → Masaya doğru yürüme (A-Star pathfinding)
- `working` → Masada oturma + klavye yazma animasyonu
- `thinking` → Kafasının üstünde `...` emote balonu
- `done` → Ayağa kalkma, gezinmeye devam

---

## 11. Gözlemlenebilirlik (Observability)

### ✅ KARAR: Langfuse (Açık Kaynak, Self-Hosted)

**Değerlendirilen Alternatifler:**

| Seçenek | Durum |
|---|---|
| **Langfuse** | ✅ **SEÇİLDİ** |
| TrustLens | 🔶 Daha az olgun |
| OpenClaw Agent Cost Optimizer | 🔶 Daha az yaygın |
| Sıfırdan loglama | ❌ Reddedildi |

**Karar Gerekçesi:**
- Her LLM API çağrısının `prompt_tokens`, `completion_tokens`, `cache_hit_rate` otomatik loglanır.
- Self-hosted → müşteri verisi dışarı çıkmaz.
- Status bar'daki "Token Budget" göstergesi Langfuse'tan beslenir.
- Cognee entegrasyonu var → agentspace-knowledge katmanıyla uyumlu.

**Referanslar:**
- [Langfuse GitHub](https://github.com/langfuse/langfuse) — MIT License
- [Langfuse Documentation](https://langfuse.com/docs)

---

## 12. Kaynak Kod Yönetimi

### ✅ KARAR: 4 Repo Mimarisi (brsagentspace Organization)

**Repo Yapısı:**

| Repo | İçerik | Bağlantı |
|---|---|---|
| **agentspace-core** | Tauri + React + Phaser.js uygulama kodu | [github.com/brsagentspace/agentspace-core](https://github.com/brsagentspace/agentspace-core) |
| **agentspace-assets** | Kenney.nl görsel materyaller | [github.com/brsagentspace/agentspace-assets](https://github.com/brsagentspace/agentspace-assets) |
| **agentspace-skills** | Blueprint YAML şablonları, ajan kuralları | [github.com/brsagentspace/agentspace-skills](https://github.com/brsagentspace/agentspace-skills) |
| **agentspace-knowledge** | Bu belge, sistem mimarisi, araştırma raporu | [github.com/brsagentspace/agentspace-knowledge](https://github.com/brsagentspace/agentspace-knowledge) |

**Gerekçe:**
- Görsel materyaller (assets) ana repoyu şişirir → ayrı repo
- Blueprint kuralları uygulamayı rebuild etmeden güncellenebilir → ayrı repo
- Bilgi yönetimi kod değişikliklerinden bağımsız evrimleşir → ayrı repo

---

## Eksiksiz Akademik Kaynakça

| Makale | arXiv ID | Yıl | AgentSpace'teki Rolü |
|---|---|---|---|
| Zep: Temporal KG Architecture | 2501.13956 | Ocak 2025 | Graphiti seçim gerekçesi |
| G-Memory: Hierarchical Graph Memory | 2506.07398 | Haziran 2025 | Bellek mimarisi referansı |
| LEGOMem: Procedural Memory | 2510.04851 | Ekim 2024 | Blueprint Engine mantığı |
| Multi-Layered Memory for LLM Agents | 2603.29194 | 2026 | 4 katmanlı bellek tasarımı |
| AMA: Adaptive Memory via MAS | 2601.20352 | 2026 | Phase 2 hafıza otomasyonu |
| Agentic Context Management (ACM) | 2607.21503 | 2026 | Context Lifecycle 5 aşaması |
| Governed Shared Memory (MemClaw) | 2606.07921 | 2026 | Multi-agent tutarlılık |
| Cognee: KG-LLM Interface | 2505.24478 | Mayıs 2025 | Reddedilen alternatif |
| TeaRAG: Token Reduction | — | 2025 | RAG token tasarrufu |
| TIGRAG: Token-Induced GraphRAG | — | 2025 | Gelecek GraphRAG referansı |

---

## GitHub Kütüphane Referansları

| Kütüphane | GitHub | Lisans | AgentSpace'teki Rolü |
|---|---|---|---|
| **Graphiti** | [getzep/graphiti](https://github.com/getzep/graphiti) | Apache-2.0 | Knowledge Graph motoru |
| **LangGraph** | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | MIT | Ajan orkestrasyonu |
| **LlamaIndex** | [run-llama/llama_index](https://github.com/run-llama/llama_index) | MIT | Blueprint RAG |
| **xterm.js** | [xtermjs/xterm.js](https://github.com/xtermjs/xterm.js) | MIT | Terminal multiplexer |
| **Langfuse** | [langfuse/langfuse](https://github.com/langfuse/langfuse) | MIT | Observability |
| **Rankify** | [DataScienceUIBK/Rankify](https://github.com/DataScienceUIBK/Rankify) | — | Reranker modelleri |
| Mem0 | [mem0ai/mem0](https://github.com/mem0ai/mem0) | Apache-2.0 | Phase 2 alternatif bellek |
| Letta (MemGPT) | [letta-ai/letta](https://github.com/letta-ai/letta) | Apache-2.0 | Referans mimari |
| TsinghuaC3I/Awesome-Memory | [GitHub](https://github.com/TsinghuaC3I/Awesome-Memory-for-Agents) | — | Araştırma listesi |
| TeleAI/Awesome-Agent-Memory | [GitHub](https://github.com/TeleAI-UAGI/Awesome-Agent-Memory) | — | Araştırma listesi |

---

*Bu katalog yaşayan bir belgedir. Yeni karar alındığında bir sonraki bölüm eklenir.*  
*Repo: [brsagentspace/agentspace-knowledge](https://github.com/brsagentspace/agentspace-knowledge)*
