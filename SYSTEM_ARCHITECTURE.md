# AgentSpace — Sistem Mimarisi ve Bağlam Tasarımı (Master Document)

> Bu belge, bir ajan bir göreve başlamadan önce ne bilmesi gerektiğini,
> çalışırken hangi verileri tüketmesi gerektiğini ve işi bitince
> sistemde ne bırakması gerektiğini en ayrıntılı şekilde tanımlar.

---

## 1. Temel Felsefe: "Context Lifecycle"

Bir ajanı bir çalışana benzetebiliriz. İyi bir çalışan işe başlamadan önce:
- Şirketin **kurallarını** (kodlama standartları, tasarım prensipleri) bilir
- Projenin **geçmişini** (önceki kararlar, hatalar, neden şöyle yapıldı) bilir
- Diğer çalışanların **ne yaptığını** (paralel işler, bağımlılıklar) bilir
- İşi bitince yaptıklarını **raporlar** (öğrenilenler, üretilen çıktılar)

```
┌─────────────────────────────────────────────────────┐
│                CONTEXT LIFECYCLE                     │
│                                                     │
│  [BAŞLAMADAN ÖNCE]   [ÇALIŞIRKEN]    [BİTİNCE]      │
│                                                     │
│  Retrieve            Inject          Persist         │
│  · Proje kuralları   · Sliding       · Karar özeti   │
│  · Blueprint         · Window        · Hata kaydı    │
│  · Geçmiş kararlar   · Checkpoint    · KB güncellemesi│
│  · Diğer ajan        · File State    · Sonraki adım  │
│    state'leri                                        │
└─────────────────────────────────────────────────────┘
```

---

## 2. Knowledge Graph — Node ve Edge Şeması

### 2.1 Node Tipleri (Varlıklar)

```yaml
nodes:
  Agent:
    description: "Sistemdeki her bir AI robotu"
    properties:
      - id: string           # "agent-backend-001"
      - name: string         # "Kral" (özel isim)
      - model: string        # "claude-sonnet-4-5"
      - role: string         # "backend | frontend | qa | researcher | marketing"
      - state: enum          # "idle | thinking | working | blocked | done"
      - current_task: ref
      - token_budget: int

  Project:
    description: "Her bir müşteri veya kişisel proje"
    properties:
      - id: string
      - name: string
      - type: enum           # "mobile | web | backend | ml | fullstack"
      - blueprint: ref
      - status: enum         # "planning | active | review | done | paused"
      - client: string
      - deadline: date

  Task:
    description: "Bir ajan tarafından yürütülen atomik iş birimi"
    properties:
      - id: string
      - title: string
      - status: enum         # "queued | assigned | in_progress | done | failed"
      - assigned_agent: ref
      - project: ref
      - priority: int        # 1-5
      - depends_on: [ref]
      - tokens_used: int
      - output_files: [path]

  Blueprint:
    description: "Bir proje tipi için metodolojik şablon"
    properties:
      - id: string
      - name: string
      - project_type: enum
      - architecture_rules: [string]
      - folder_structure: yaml
      - design_principles: [string]   # SÜREÇ bazlı, 1-1 kopya değil
      - agent_roles: yaml
      - reference_projects: [ref]     # Sadece süreç referansı
      - forbidden_patterns: [string]

  Decision:
    description: "Neden X yerine Y seçildi — ajanların öğrenmesi için"
    properties:
      - id: string
      - project: ref
      - context: string      # "State management için ne kullanacağız?"
      - chosen: string       # "Zustand"
      - rejected: [string]   # ["Redux", "Jotai"]
      - reason: string
      - made_by: string      # "human" | agent id
      - timestamp: datetime
      - outcome: enum        # "successful | failed | revised"

  Lesson:
    description: "Bir hatadan veya başarıdan çıkarılan öğrenim"
    properties:
      - id: string
      - project: ref
      - type: enum           # "error | optimization | pattern | anti-pattern"
      - summary: string
      - applies_to: [ref]    # Hangi blueprint'e uygulanır
      - severity: enum       # "critical | major | minor | tip"
```

### 2.2 Edge Tipleri (İlişkiler)

```yaml
edges:
  AGENT_WORKS_ON:        Agent → Task
  AGENT_BLOCKED_BY:      Agent → Task (with: reason, since)

  PROJECT_USES_BLUEPRINT: Project → Blueprint
  PROJECT_INSPIRED_BY:    Project → Project
    note: "Sadece SÜRECİ referans alır, görselliği değil"
    properties: [aspect: "architecture | testing | design_process"]

  TASK_DEPENDS_ON:       Task → Task (type: "blocks | informs | parallel")
  DECISION_IN_PROJECT:   Decision → Project
  LESSON_UPGRADES:       Lesson → Blueprint  # Hatalar blueprint'e kural olarak eklenir
```

---

## 3. Agent Feed Pipeline — Bir Ajan İşe Başlamadan Önce Ne Yemeli?

Ajana HER ŞEYİ vermiyoruz. Sadece **o an için gerekli katmanı** inject ediyoruz.

```
KATMAN 0: System Prompt (SABİT — Provider Cache'i Devreye Girer)
──────────────────────────────────────────────────────────────
· Ajanın kimliği ve rolü
· Evrensel kurallar (asla ihlal edilmez)
· Çıktı formatı kuralları
· Hangi araçları kullanabileceği
⚡ Bu katman HİÇ DEĞİŞMEZ → %50-90 token tasarrufu

KATMAN 1: Blueprint Bağlamı (Proje tipine göre seçilir)
───────────────────────────────────────────────────────
· Mimari kurallar ve klasör yapısı
· Tasarım süreci prensipleri
· Yasaklı kalıplar

KATMAN 2: Proje Bağlamı (Sıkıştırılmış)
─────────────────────────────────────────
· Projenin hedefi (1-2 paragraf özet)
· Alınan kritik kararlar (Decision node'ları)
· Bilinen kısıtlamalar

KATMAN 3: Görev Bağlamı (O anki iş)
─────────────────────────────────────
· Görevin tam tanımı
· Bağımlı görevlerin ÖZET çıktıları
· Sadece ilgili dosyaların içerikleri

KATMAN 4: Epistodik Bağlam (Dinamik)
──────────────────────────────────────
· Diğer ajanların anlık state.json'ları
· Son checkpoint dosyaları
```

### 3.1 Token Bütçesi ve Model Seçimi

| Tier | Görev | Model | Max Context |
|---|---|---|---|
| **Tier 1** | Mimari karar, karmaşık kod | Claude Sonnet / GPT-4o | 180k |
| **Tier 2** | Rutin kod, refactoring, dokümantasyon | Claude Haiku / GPT-4o-mini | 60k |
| **Tier 3** | Özetleme, log analizi, yönlendirme | Gemini Flash | 20k |
| **Tier 4** | Embedding, RAG sorgusu, gizlilik | Yerel model (Nomic vb.) | — |

---

## 4. Bellek Mimarisi (4 Katman)

```
L1 — ÇALIŞMA BELLEĞİ (Working Memory)
   · Aktif görev penceresi, son N mesaj
   · Süre: O oturum
   · Yönetim: Sliding window + otomatik sıkıştırma

L2 — EPİZODİK BELLEĞİ (Session Summary)
   · Tier-3 model ile özetlenen oturum kaydı
   · "Bu oturumda ne yapıldı, ne karar alındı"
   · Format: progress.md (versiyonlanmış, Git'te)

L3 — SEMANTİK BELLEĞİ (Knowledge Graph)
   · Kalıcı kararlar, dersler, kural güncellemeleri
   · RAG ile sorgulanır
   · Format: JSON / YAML (Git'te versiyonlanır)

L4 — PROSEDÜREL BELLEĞİ (Blueprint + Rules)
   · "Nasıl yapılır" bilgisi
   · Zaman içinde Lesson node'larıyla evrimleşir
   · Kalıcı ve büyüyen bir yapı
```

### 4.1 Context Rot Önleme

Context penceresi %70'i geçince **Tier-3 model** devreye girer ve sıkıştırır.

```yaml
promotion_filter:
  # Sadece bunlar L3'e (Knowledge Graph) gider:
  - type: Decision  →  outcome confirmed
  - type: Lesson    →  severity >= major
  - type: Pattern   →  reusable == true
  # Ham konuşma geçmişi ASLA L3'e gitmez
```

---

## 5. Blueprint Engine — Metodolojik Şablon Yapısı

```
agentspace-skills/
└── blueprints/
    ├── _base.yaml                    # Tüm şablonların miras aldığı
    ├── mobile-react-native.yaml
    ├── web-nextjs-fullstack.yaml
    ├── backend-node-microservice.yaml
    ├── backend-rust-service.yaml
    └── ml-python-pipeline.yaml
```

### 5.1 Örnek Blueprint YAML

```yaml
id: "blueprint-mobile-rn-v2"
name: "React Native Mobil Projesi"
project_type: "mobile"

architecture:
  pattern: "Feature-Sliced Design"
  state_management: "Zustand (global) + React Query (server state)"
  navigation: "Expo Router"
  forbidden:
    - "Context API'yi global state için kullanma"
    - "useEffect içinde business logic yazma"
    - "any tipi kullanma (TypeScript)"

design_process:
  # UYARI: Bu kurallar SÜREÇ kurallarıdır. 
  # Referans projenin rengi, görseli, UI'si 1-1 kopyalanmaz.
  # Her proje görsel olarak tamamen özgün olacak.
  principles:
    - "Atomic Design: atom > molecule > organism > template > page"
    - "8pt grid sistemi (tüm spacing 8'in katı)"
    - "Yeni proje = özgün renk paleti. Referansı süreçten al, görselden değil."
    - "Tipografi: max 3 farklı font weight"
    - "Touch target: min 44x44pt"

agent_roles:
  architect:   { count: 1, tier: 1 }
  frontend:    { count: 2, tier: 2 }  # Paralel çalışabilir
  qa:          { count: 1, tier: 2 }
  reviewer:    { count: 1, tier: 3 }
```

---

## 6. State File Protocol — Bağlam Kopukluğunu Önleme

```json
// .agentspace/agents/agent-backend-001/state.json
{
  "agent_id": "agent-backend-001",
  "project_id": "proj-ecommerce-2025",
  "task_id": "task-api-auth",

  "completed_steps": [
    "User model oluşturuldu",
    "JWT middleware yazıldı"
  ],
  "current_step": "Refresh token mekanizması yazılıyor",
  "next_steps": [
    "Register endpoint",
    "Password reset flow"
  ],

  "key_decisions": [
    "Redis yerine in-memory token store (ilk faz için)",
    "Bcrypt cost factor 12"
  ],

  "files_created": [
    "src/features/auth/user.model.ts",
    "src/features/auth/jwt.middleware.ts"
  ],

  "tokens_used": 12450,
  "tokens_remaining": 27550
}
```

Context sıfırlansa bile ajan bu dosyayı okuyarak **saniyeler içinde** tam bağlamını yeniden kurar.

---

## 7. Orkestrasyon Akışı

```
SİZ: "Yeni mobil proje, X müşterisi, e-ticaret"
          │
          ▼
    [ORCHESTRATOR]
    · Blueprint seç: mobile-rn
    · Proje node'u oluştur
    · Görev listesini çıkar
    · Ajan rollerini ata
          │
    ┌─────┴──────────┐
    │                │
    ▼                ▼
[Ajan-1: Architect] [Ajan-2: Frontend]
Feed: L0+L1+L2+L3   Feed: L0+L1+L2+L3+Ajan-1 output
    │                │
    ▼                ▼
Decision node    File node'ları
oluşur           oluşur
    │                │
    └────────────────┘
              │
              ▼
       Knowledge Graph
       Güncellenir (L3)
              │
              ▼
       Lesson varsa Blueprint'e
       kural olarak eklenir (L4)
```

---

## 8. Muratify'dan Farklılaştığımız 5 Kritik Nokta

| Özellik | Muratify AgentSpace | Bizim Sistemimiz |
|---|---|---|
| **Bellek** | Oturum bazlı, kaybolur | 4 katmanlı kalıcı mimari |
| **Bağlam** | Kapalı kutu | Açık YAML/JSON, Git'te versiyonlanır |
| **Şablonlar** | Yok | Blueprint Engine (metodoloji bazlı) |
| **Token verimliliği** | Bilinmiyor | Tier sistemi + cache + compaction |
| **Öğrenme** | Yok | Lesson → Blueprint döngüsü |

---

## 9. Açık Sorular (Netleştirilmesi Gereken)

> [!IMPORTANT]
> **Ek Blueprint Tipleri:** Şu an 5 tip planlandı. "Design-only" veya "DevOps/Infrastructure" projeleri eklensin mi?

> [!IMPORTANT]
> **Graph Motoru:** Başlangıçta Git'teki YAML dosyaları yeterli. İleride Neo4j gibi bir graph DB'ye geçmek ister misiniz?

> [!IMPORTANT]
> **Ajan İsimlendirme:** Soyut ID mi (agent-backend-001) yoksa ofis atmosferi için özel isimler mi (Kral, Prens)?

> [!IMPORTANT]
> **Sesli Komut (AgentVoice):** MVP'de mi olsun yoksa Phase 2'ye mi atılsın?

> [!IMPORTANT]
> **Marketing Tetikleyici:** Proje "done" olunca marketing ajanı otomatik mı devreye girsin?
