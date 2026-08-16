# AgentSpace — Akademik Araştırma ve Kütüphane Raporu (2025-2026)

> Sıfırdan inşa etmek yerine hangi savaşta durulacağını bilmek asıl zekadır.
> Bu belge, mevcut en iyi açık kaynak kütüphaneleri ve 2025-2026 akademik
> bulgularını değerlendirerek AgentSpace için nihai teknoloji seçimini ortaya koyar.

---

## 1. Akademik Makaleler (2025-2026)

### 1.1 G-Memory — Hiyerarşik Graf Hafıza (Haziran 2025)
**Kaynak:** arXiv:2506.07398 | **GitHub:** `bingreeky/GMemory`

En doğrudan bizimle örtüşen araştırma. Çoklu ajan sistemleri için
3 katlı bir Graf hiyerarşisi önerir:

```
Katman 3 (En Üst): INSIGHT GRAPH
  └─ Geçmiş deneyimlerden çıkarılan genel dersler
  └─ "Bu tip görevde X yaklaşımı %80 başarılı oldu"

Katman 2: QUERY GRAPH
  └─ Daha önce çözülen/başarısız olan görevler
  └─ Status: Resolved / Failed
  └─ Semantic ilişkilerle birbirine bağlı

Katman 1 (En Alt): INTERACTION GRAPH
  └─ Ajan-ajan iletişim logları
  └─ Anlık görev yürütme trafiği
```

**Bulgu:** AutoGen, DyLAN ve MacNet üzerinde test edildi.
Başarı oranını benchmark'larda anlamlı şekilde artırdı.

**AgentSpace için önemi:**
Bizim Knowledge Graph mimarimiz bu 3 katmanla mükemmel örtüşüyor.
"Lesson node" (bizim L4) = Insight Graph.
"Decision node" (bizim L3) = Query Graph.
"State file" (bizim L1) = Interaction Graph.

> ✅ **Karar: Mimarimiz zaten bununla uyumlu. Uygulamada referans alınacak.**

---

### 1.2 LEGOMem — Prosedürel Bellek Modülleri (Ekim 2024 → 2025 update)
**Kaynak:** arXiv:2510.04851

Çoklu ajan sistemlerinde "stateless" problemi çözer. Başarılı
görev izlerini (execution traces) yeniden kullanılabilir bellek
birimlerine (reusable memory units) dönüştürür.

**En kritik buluş:**
```
Orkestratör için → Full-task memories (üst düzey planlama)
Alt ajanlar için → Subtask memories (spesifik araç kullanımı)
```

OfficeBench benchmark'ında **+12-13 puan** başarı artışı.

**AgentSpace için önemi:**
Blueprint Engine'imizin "Lesson" → "Blueprint'e kural ekle" döngüsü
tam olarak bu mantığı uygulıyor. Fark şu: LEGOMem bunu vektör DB'de
tutuyor, biz YAML'da tutuyoruz.

> ✅ **Karar: LEGOMem'in retrieval stratejisini Blueprint Engine'e entegre edeceğiz.**

---

### 1.3 AMA — Adaptive Memory via Multi-Agent Collaboration (2026)
**Kaynak:** arXiv:2601.20352

4 ajanın hafızayı birlikte yönettiği bir sistem:
- **Constructor:** Ham veriden hafıza birimi oluşturur
- **Retriever:** İlgili hafızayı bulur ve getirir
- **Judge:** Hafızanın kalitesini değerlendirir
- **Refresher:** Eski/hatalı hafızayı günceller veya siler

**AgentSpace için önemi:**
Şu an "Lesson" node'larına biz manuel olarak ne yazılacağına
karar veriyoruz. AMA mantığı bu süreci otomasyona alır.
Ajan kendi hafızasını değerlendirecek bir "Judge" bileşenine sahip olur.

> 🔶 **Karar: Phase 2 için referans. MVP'de manuel yönetim yeterli.**

---

### 1.4 Agentic Context Management (ACM) (2026)
**Kaynak:** arXiv:2607.21503

Context'i "depolama" değil "yaşam döngüsü" olarak tanımlayan ilk formal çalışma.
5 aşama önerir: **Architecting → Ingesting → Scoping → Anticipating → Compacting**

"Anticipating" kavramı özellikle ilginç: Ajan bir sonraki adımda
ne tüketecekse onu önceden hazırlar (prefetch pattern).

> ✅ **Karar: Context Lifecycle mimarimizi bu 5 aşamayla rafine edeceğiz.**

---

## 2. Kütüphane Karşılaştırması

### 2.1 Ana Rakipler

| Kütüphane | Stars | Lisans | Bellek Modeli | Güçlü Yanı |
|---|---|---|---|---|
| **Mem0** | ~48K | Apache-2.0 | Hybrid vector+graph+KV | Rapid prototype, personalization |
| **Cognee** | ~5K | Apache-2.0 | Knowledge Graph (ECL pipeline) | Yapısal hafıza, self-hosted |
| **Letta (MemGPT)** | ~35K | Apache-2.0 | OS-style tiered (RAM/disk) | Uzun vadeli ajan, otonom bellek |
| **Graphiti (Zep)** | ~7K | Apache-2.0 | Bi-temporal knowledge graph | Temporal reasoning, enterprise |
| **LangMem** | — | MIT | Episodic + semantic + procedural | LangGraph entegrasyonu |

---

### 2.2 Derinlemesine Analiz

#### 🏆 Graphiti (getzep/graphiti) — En Güçlü Aday

**Neden öne çıkıyor:**
Tek bir **bi-temporal model** kullanır. İki zaman boyutu izler:
- **Valid Time:** Gerçekte ne zaman doğruydu
- **Ingestion Time:** Sisteme ne zaman öğretildi

```python
# Örnek: "Proje X aktif durumda" kararı
{
  "fact": "Proje X aktif",
  "valid_from": "2025-08-01",
  "valid_until": "2025-08-15",  # Proje askıya alındı
  "ingested_at": "2025-08-16"
}
```

Ajan "Bu proje geçen hafta aktif miydi?" diye sorabilir.
Knowledge Graph'te "Proje X" node'u hem geçmiş hem mevcut durumu taşır.

**Neo4j, FalkorDB, Kuzu ve Amazon Neptune ile çalışır.**
Paper: *"Zep: A Temporal Knowledge Graph Architecture for Agent Memory"* (arXiv:2501.13956, Ocak 2025)

> 🌟 **AgentSpace için: Decision node'larımızda "bu karar ne zaman verildi, hâlâ geçerli mi?" sorusu Graphiti'nin bi-temporal modeliyle doğrudan yanıtlanabilir.**

---

#### 🏆 Cognee (topoteretes/cognee) — Knowledge Graph Odaklı Seçenek

**ECL Pipeline:**
```
Extract → Cognify → Load
   │           │        │
Ham veri    Yapısal   RAG-ready
          KG haline   Knowledge
           getirilir    Graph
```

**Güçlü yanları:**
- Self-hosted (yerel, gizlilik sorunsuz)
- LanceDB ile dosya tabanlı çalışır (kurulum karmaşıklığı yok)
- Apache-2.0 lisanslı, tamamen açık kaynak
- Langfuse ile observability entegrasyonu var

**Zayıf yanı:** Graphiti kadar güçlü temporal reasoning yok.

> ✅ **AgentSpace için: Knowledge Graph veri katmanı olarak güçlü aday.**

---

#### Mem0 (mem0ai/mem0) — Hız ve Kolaylık

48K yıldız, en büyük topluluk. "Drop-in" memory layer.
Hybrid: vector + graph + key-value.

**Neden birincil seçim değil:**
- Temporal reasoning zayıf (Graphiti kadar güçlü değil)
- Self-hosted versiyon cloud versiyonu kadar güçlü değil
- Graph tarafı hâlâ gelişmekte

> 🔶 **Rol: Hızlı prototipleme aşamasında veya kullanıcı bazlı kişiselleştirme için.**

---

#### Letta (letta-ai/letta) — "Memory OS"

MemGPT'nin yeniden markalaşması. Ajan kendi belleğini yönetir,
RAM'dan disk'e "paging" yapar.

**Güçlü yanı:** Otonom, agent-managed memory.
**Zayıf yanı:** Bizim sistemimizde belleği ajana değil, orkestratöre
bırakmak istiyoruz. Letta belleği tam ajan kontrolüne verir.

> 🔶 **Rol: İleride bir ajana "kendi hafızasını yönet" görevi verilirse.**

---

### 2.3 MCP (Model Context Protocol) Durumu

2026'da MCP tamamen **stateless** bir standarda evrildi.
Artık persistence dışarıya (SQLite, Postgres, vector DB) taşınıyor.

**AgentSpace için kritik çıkarım:**
State file'larımızı (`state.json`) MCP Filesystem Server üzerinden
standart bir araç (tool) olarak expose edebiliriz. Her ajan, kendi
state dosyasını MCP tool çağrısıyla okur/yazar. Bu sayede
Tauri arka planı ile LLM katmanı arası temiz bir köprü kurulur.

---

## 3. Nihai Teknoloji Seçim Matrisi

```
┌─────────────────────────────────────────────────────────────────┐
│              AGENTSPACE TEKNOLOJİ KARARI                        │
│                                                                 │
│  KATMAN          NE KULLANALIM?          NEDEN?                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  L1 Çalışma      State File (JSON)       MCP FS üzerinden      │
│  Belleği         + Sliding Window        expose edilir         │
│                                                                 │
│  L2 Epizodik     progress.md             Git'te versiyon-      │
│  Bellek          (Tier-3 model özetler)  lanmış, insan okur    │
│                                                                 │
│  L3 Semantik     Graphiti (Açık Kaynak)  Bi-temporal KG,       │
│  Bellek          veya Cognee             self-hosted, Apache    │
│  (Knowledge                              2.0                    │
│  Graph)                                                         │
│                                                                 │
│  L4 Prosedürel   Blueprint YAML          LEGOMem mantığı,      │
│  Bellek          (Git'te yaşar)          Lesson → Blueprint     │
│                  + LEGOMem retrieval     döngüsü                │
│                                                                 │
│  Orchestration   LangGraph               Proven, production-   │
│                                          ready, LangMem uyumu  │
│                                                                 │
│  Observability   Langfuse                Token tracking,        │
│                  (açık kaynak)           cost monitoring        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Sıfırdan Yapılacaklar vs. Mevcut Kütüphane

| Bileşen | Yaklaşım | Gerekçe |
|---|---|---|
| **Knowledge Graph motoru** | **Graphiti veya Cognee** (hazır) | Bi-temporal KG sıfırdan 6 ay sürer |
| **Context lifecycle** | **LangGraph** (hazır) | Proven, enterprise-grade |
| **Token observability** | **Langfuse** (hazır) | Açık kaynak, self-hosted |
| **Blueprint Engine** | **Sıfırdan (YAML+Python)** | Sisteme özgü, hazır yok |
| **State File Protocol** | **Sıfırdan (JSON+MCP)** | Sisteme özgü |
| **Orchestration UI** | **Sıfırdan (Tauri+React)** | Tam özel |
| **2D Ofis Simülasyonu** | **Sıfırdan (Phaser.js)** | Sisteme özgü |
| **Agent Tier Router** | **Sıfırdan (rule-based)** | Sisteme özgü, basit |

---

## 5. Önerilen Karar

### Graphiti → Knowledge Graph Motorumuz Olmalı

**Gerekçe:**
1. Apache-2.0 lisanslı, tamamen açık kaynak
2. Self-hosted (yerel, gizlilik sorunu yok)
3. Bi-temporal model → "Proje X geçen ay aktif miydi?" sorusunu yanıtlayabilir
4. Neo4j veya FalkorDB backend'i (yerel kurulumda FalkorDB daha hafif)
5. Var olan Knowledge Graph şemasımızla uyumlu
6. **2025 arxiv makalesiyle akademik temeli kanıtlanmış**

### LangGraph → Orkestrasyon katmanı

Ajan state machine'ini LangGraph ile yönetirsek:
- Her ajan = bir node
- Her ajan geçişi = bir edge
- Context lifecycle = LangGraph checkpointer

### Langfuse → Token ve maliyet takibi

Her API çağrısının token sayısını, model tierni ve maliyetini
otomatik loglar. Status bar'daki "Token Budget" göstergesi
buradan beslenir.

---

## 6. Eylem Planı

```
HEMEN:
[ ] Graphiti GitHub'ı incele: getzep/graphiti
[ ] Cognee ile Graphiti arasında final karar
[ ] LangGraph'in checkpointer sistemini incele

SONRA:
[ ] agentspace-skills/blueprints/ altında blueprint YAML'ları yaz
[ ] Graphiti ile Knowledge Graph node şemasını oluştur
[ ] LangGraph ile orkestratör agent'ı prototiple

KODLAMAYA SON:
[ ] Tauri + React scaffold
[ ] Phaser.js ofis simülasyonu
[ ] xterm.js terminal entegrasyonu
```

---

## 7. Akademik Referans Listesi

| Paper | arXiv | Yıl | Kullanım Alanı |
|---|---|---|---|
| G-Memory | 2506.07398 | 2025 | KG mimarisi referansı |
| LEGOMem | 2510.04851 | 2024-2025 | Blueprint Engine mantığı |
| AMA | 2601.20352 | 2026 | Phase 2 hafıza otomasyonu |
| ACM (Maximem) | 2607.21503 | 2026 | Context Lifecycle 5 aşaması |
| Governed Shared Mem | 2606.07921 | 2026 | Multi-agent bellek tutarlılığı |
| Graphiti/Zep | 2501.13956 | 2025 | Knowledge Graph motoru |

---

## 8. Curated GitHub Listeleri (Okuma Listesi)

- [TsinghuaC3I/Awesome-Memory-for-Agents](https://github.com/TsinghuaC3I/Awesome-Memory-for-Agents) — Kapsamlı akademik liste
- [TeleAI-UAGI/Awesome-Agent-Memory](https://github.com/TeleAI-UAGI/Awesome-Agent-Memory) — Fonksiyonelite bazlı sıralama
- [getzep/graphiti](https://github.com/getzep/graphiti) — Knowledge Graph motoru
- [topoteretes/cognee](https://github.com/topoteretes/cognee) — ECL pipeline KG

---

*Bu araştırma raporu `agentspace-knowledge` reposuna commit edilecek.*
*Her yeni keşif bu belgeye eklenir.*
