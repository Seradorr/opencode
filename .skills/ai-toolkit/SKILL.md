---
name: ai-toolkit
description: |
  Acik kaynak AI kodlama asistani kurulumu, model secimi, system prompt
  muhendisligi, agent konfigurasyonu ve OpenCode/Aider/Continue.dev
  optimizasyonu icin kullan.
---

# ACIK KAYNAK AI KODLAMA ASISTANI KURULUM VE KONFIGURASYON REHBERI

Bu skill, acik kaynak AI araclarini kullanarak Codex, Claude Code veya
GitHub Copilot seviyesinde calisacak bir kodlama asistani olusturma
konusunda uzmanlik saglar.

---

## 1. ARAC EKOSISTEMI

### Model Servisleri (Backend)

| ARAC | KULLANIM | KURULUM |
|------|----------|---------|
| Ollama | Yerel model calistirma, kolay API | `curl -fsSL https://ollama.com/install.sh \| sh` |
| vLLM | Yuksek performans, production API | `pip install vllm` |
| llama.cpp | Dusuk kaynak, GGUF modeller | Kaynak derle veya release indir |
| LM Studio | GUI tabanli, kolay kullanim | Installer indir |
| text-generation-webui | Esnek, cok model destegi | `git clone + pip install` |
| LocalAI | OpenAI uyumlu API, konteyner | `docker run localai/localai` |
| TabbyML | Kod tamamlama odakli | `docker run tabbyml/tabby` |

### Kodlama Asistan Arayuzleri (Frontend)

| ARAC | ENTEGRASYON | EN IYI ICIN |
|------|-------------|-------------|
| OpenCode | Terminal CLI | Tam agent, tool use, multi-agent |
| Aider | Terminal CLI | Git-aware kod duzenleme |
| Continue.dev | VS Code eklentisi | IDE icinde chat + autocomplete |
| Tabby | VS Code / JetBrains | Kod tamamlama (self-hosted) |
| Open Interpreter | Terminal CLI | Sistem erisimli agent |
| Cursor | IDE (fork) | Tam IDE deneyimi |
| Cline | VS Code eklentisi | Agent modu, tool use |
| avante.nvim | Neovim eklentisi | Neovim kullanicilari |

---

## 2. MODEL SECIM REHBERI

### Mevcut Modeller (OpenAI Uyumlu API)

Tum modeller OpenAI uyumlu API uzerinden erisime aciktir. Qwen modeller 256K
context ve 64K output ile kullanilir; diger model limitleri provider tanimina
gore ayarlanir.

| MODEL | CONTEXT | OZELLIKLER | EN IYI ICIN |
|-------|---------|------------|-------------|
| Qwen3.5-397B-A17B-FP8 | 256K / 64K output | Thinking, Tool Use, Image | En zor gorevler, mimari analiz, plan |
| Kimi-K2.5 | 128K | Thinking, Tool Use, Image | Reasoning, uzun metin, dokumantasyon |
| GLM-4.7-FP8 | 128K | Thinking, Tool Use | Alternatif reasoning, test yazimi |
| Qwen3.5-35B-A3B-FP8 | 256K / 64K output | Thinking, Tool Use, Image | Hizli cevap, basit isler, gunluk kodlama |

### Agent-Model Eslestirme

| AGENT | MODEL | NEDEN |
|-------|-------|-------|
| build | Qwen3.5-397B-A17B-FP8 | En guclu model, karmasik kodlama |
| plan | Qwen3.5-397B-A17B-FP8 | Thinking modu, derin analiz |
| review | Qwen3.5-397B-A17B-FP8 | Detayli kod inceleme |
| refactor | Qwen3.5-397B-A17B-FP8 | Buyuk resmi gorme, optimizasyon |
| docs | Kimi-K2.5 | Uzun metin uretimi, tutarli cikti |
| test | GLM-4.7-FP8 | Reasoning ile edge-case tespiti |
| fast | Qwen3.5-35B-A3B-FP8 | Hizli, dusuk latency |

### Context Length Stratejisi

Qwen modellerde 256K context mevcuttur. Ancak her istekte tum pencereyi
doldurmak maliyeti arttirir ve hizi dusurur. Varsayilan kullanimda ilgili
baglam kadar pencere ayrilmali, gerekli oldugunda genisletilmelidir.

| SENARYO | ONERILEN CONTEXT | NEDEN |
|---------|------------------|-------|
| Tekli dosya chat | 8K-16K | Cogu dosya yeterli |
| Multi-dosya agent | 32K-64K | Buyuk codebase, standart kullanim |
| Ozel analiz | 64K-256K | Gerektiginde model limiti arttirilabilir |

---

## 3. KONFIGURASYON OPTIMIZASYONU

### Temel Parametreler

```jsonc
{
  // Model servisi ayarlari
  "temperature": 0.7,        // Kodlama icin 0.3-0.8 arasi ideal
  "top_p": 0.9,              // Nucleus sampling
  "max_tokens": 8192,        // Cikti limiti - gorev bazli ayarla
  "frequency_penalty": 0.0,  // Tekrar onleme (genelde 0)
  "presence_penalty": 0.0,   // Konu cesitliligi (genelde 0)
  "stop": ["```\n", "\n\n\n"] // Gereksiz devami kes
}
```

### Gorev Bazli Temperature Ayarlari

| GOREV | TEMPERATURE | NEDEN |
|-------|-------------|-------|
| Bug fix / debug | 0.2 - 0.4 | Az yaraticilik, cok dogruluk |
| Kod inceleme (review) | 0.3 | Tutarli, detayli analiz |
| Test yazma | 0.4 | Edge-case tespiti dengeli |
| Refactoring | 0.3 - 0.5 | Dengeli |
| Mimari planlama | 0.5 | Derin dusunme, tutarli cikti |
| Teknik dokumantasyon | 0.6 | Acik ve tutarli |
| Kod yazma (build) | 0.7 | Yaratici cozumler |
| Hizli gorevler (fast) | 0.7 | Gunluk is, esneklik |

### OpenCode icin opencode.jsonc Sablonu

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 10000
  },
  "provider": {
    "custom": {
      "name": "CustomModels",
      "npm": "@ai-sdk/openai-compatible",
      "models": {
        "Qwen3.5-397B-A17B-FP8": {
          "name": "Qwen Hard",
          "limit": { "context": 262144, "output": 65536 },
          "options": { "top_p": 0.8 }
        },
        "Kimi-K2.5": {
          "name": "Kimi",
          "limit": { "context": 131072, "output": 32768 },
          "options": { "top_p": 0.8 }
        },
        "GLM-4.7-FP8": {
          "name": "GLM Hard",
          "limit": { "context": 131072, "output": 32768 }
        },
        "Qwen3.5-35B-A3B-FP8": {
          "name": "Qwen Fast",
          "limit": { "context": 262144, "output": 65536 },
          "options": { "top_p": 0.8 }
        }
      },
      "options": {
        "baseURL": "https://api.example.com/v1"
      }
    }
  },
  "instructions": ["./.opencode/*.md"],
  "mcp": {
    "filesystem": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-filesystem",
                  "C:/Projeler"],
      "enabled": true
    },
    "memory": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-memory"],
      "enabled": true
    }
  },
  "agent": {
    "build": {
      "model": "custom/Qwen3.5-397B-A17B-FP8",
      "temperature": 0.7,
      "prompt": "./.opencode/agents/build.md",
      "tools": { "all": true }
    },
    "plan": {
      "model": "custom/Qwen3.5-397B-A17B-FP8",
      "temperature": 0.6,
      "prompt": "./.opencode/agents/plan.md",
      "tools": { "all": true }
    },
    "fast": {
      "model": "custom/Qwen3.5-35B-A3B-FP8",
      "temperature": 0.7,
      "tools": { "all": true }
    }
  }
}
```

---

## 4. SYSTEM PROMPT MUHENDISLIGI

### Etkili System Prompt Yapisi

Bir kodlama asistaninin system prompt'u su katmanlardan olusmalidir:

```
[1. ROL TANIMI]        → Kim oldugunu belirle
[2. YETENEKLER]        → Neler yapabilecegini listele
[3. KURALLAR]          → Uymasi gereken sinirlar
[4. FORMAT]            → Cikti formati
[5. TOOL USE]          → Arac kullanim talimatlari
[6. GUVENLIK]          → Sinirlamalar ve guvenlik kurallari
```

### Production-Grade Base Prompt Sablonu

```markdown
# ROL
Sen uzman bir yazilim muhendisisin. [DILLER] dillerinde ve [FRAMEWORKLER]
frameworklerinde derin bilgiye sahipsin.

# YETENEKLER
- Kod yazma, debug etme, refactoring
- Mimari tasarim ve teknik karar alma
- Test yazma ve kod incelemesi
- Teknik dokumantasyon

# KURALLAR
1. Her zaman calisan, test edilebilir kod uret
2. Mevcut kodu anlamadan degistirme
3. Kullanicinin sordugu seye odaklan, fazlasini yapma
4. Guvenlik aciklari olusturma (OWASP Top 10)
5. Magic number kullanma, sabitleri tanimla
6. Hata durumlarini handle et
7. [PROJE SPESIFIK KURALLAR]

# FORMAT
- Kod bloklari icin dil belirt: ```python
- Degisiklik yaptiginda once-sonra goster
- Dosya yollarini tam ver

# TOOL USE
Araclar kullanirken:
- Dosyayi okumadan duzenleme yapma
- Birden fazla bagimsiz arama paralel yap
- Hata alirsan farkli strateji dene

# GUVENLIK
- Kimlik bilgileri veya API anahtarlari uretme
- Destructive komutlari onaysiz calistirma
- Kullanici verisini loglamaya calisma
```

### Agent Prompt Sablonu (OpenCode / Aider / Continue)

```markdown
# [AGENT_ADI] Agent

## Gorev
[Agentin ne yaptiginin tek cumlede ozeti]

## Uzmanlik Alanlari
- [Alan 1]: [Detay]
- [Alan 2]: [Detay]

## Calisma Prensibi
1. Oncelikle mevcut kodu oku ve anla
2. Degisiklik planini acikla
3. Minimal ve hedefli degisiklik yap
4. Degisikligi dogrula

## Cikti Formati
[Nasil cevap verecegini belirle]

## Sinirlamalar
- [Yapamayacagi seyler]
- [Kullaniciya sormasi gereken durumlar]
```

---

## 5. KURULUM VE KULLANIM

### OpenCode Kurulum (Windows)

```powershell
# Scoop ile
scoop install opencode

# veya Chocolatey ile
choco install opencode

# veya npm ile
npm install -g opencode-ai

# Proje dizininde baslat
cd C:\Projeler\MyProject
opencode

# Ilk seferde /init ile proje analizi yap
# /init → AGENTS.md olusturur
```

### Continue.dev Kurulum (VS Code)

```jsonc
// Continue.dev ayarlarinda OpenAI uyumlu API'yi ekle:
// Provider: OpenAI Compatible
// Base URL: https://api.example.com/v1
// Model: Qwen3.5-397B-A17B-FP8 veya Qwen3.5-35B-A3B-FP8
```

---

## 6. MULTI-AGENT MIMARI

### Agent Rolleri

| AGENT | ROL | MODEL | TEMP |
|-------|-----|-------|------|
| build | Kod yazma, debug, yeni feature | Qwen3.5-397B | 0.7 |
| plan | Mimari analiz, sistem tasarimi | Qwen3.5-397B | 0.5 |
| review | Kod inceleme, best practice | Qwen3.5-397B | 0.3 |
| refactor | Optimizasyon, modernizasyon | Qwen3.5-397B | 0.8 |
| docs | Dokumantasyon, README | Kimi-K2.5 | 0.6 |
| test | Test yazma, testbench | GLM-4.7-FP8 | 0.4 |
| fast | Hizli gorevler, basit isler | Qwen3.5-35B | 0.7 |

### Model Strateji

```
Qwen3.5-35B-A3B-FP8   → Hizli cevap, basit isler, gunluk kodlama
GLM-4.7-FP8       → Test yazma, reasoning gerektiren edge-case analizi
Kimi-K2.5         → Uzun metin, dokumantasyon, derin reasoning
Qwen3.5-397B      → Karmasik kodlama, mimari kararlar, ana agent
```

---

## 7. TOOL USE KONFIGURASYONU

### OpenCode Tool Tanimlari

```jsonc
{
  "tools": {
    "all": true
  }
}
```

### MCP (Model Context Protocol) Nedir?

MCP, AI asistanlarinin dis sistemlere (dosya sistemi, veritabani, API, araclar)
erisimini standartlastiran acik bir protokoldur. AI modeli dogrudan disk okuyamaz
veya veritabanina erisemez — MCP server'lar bu kopruyu kurar.

```
[AI Asistan] ←→ [MCP Client] ←→ [MCP Server] ←→ [Dis Sistem]
                                      ↓
                              Dosya sistemi, DB,
                              API, Git, terminal...
```

**Neden kullanilir?**
- AI'a guvenlice dosya okuma/yazma yetkisi verir
- Veritabanini sorgulayip sonuclari AI'a iletir
- Tek bir standartla birden fazla araca baglanir
- Her arac icin ayri entegrasyon yazmaya gerek kalmaz

### Faydali MCP Server'lar

Farkli ortamlarda kullanilabilecek MCP server'lar:

| MCP SERVER | NE YAPAR | KULLANIM |
|------------|----------|----------|
| filesystem | Dosya okuma/yazma/arama | Proje dosyalarina guvenli erisim |
| sqlite / postgres | Veritabani sorgulama | Yerel DB'leri AI ile sorgula |
| git | Git islemleri (log, diff, blame) | Yerel repo analizi |
| memory | Kalici hafiza | Agent'a uzun sureli bellek ver |
| sequential-thinking | Adim adim dusunme | Karmasik problemlerde reasoning |
| puppeteer/playwright | Web otomasyon | Web uygulamalarini test et |
| fetch | HTTP istekleri | REST API'lere erisim |

### MCP Konfigurasyon Ornegi

```jsonc
{
  "mcp": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "C:/Projects", "C:/Docs"]
    },
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite",
               "C:/Data/project.db"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

> **Not:** MCP server npm paketleri ilk kurulumda internet gerektirir.
> Sonrasinda yerel cache'ten calisir. Offline ortam icin paketleri
> onceden `npm pack` ile indirip yerel kurulum yapilabilir.

---

## 8. PERFORMANS OPTIMIZASYONU

### VRAM Hesaplama Formulu

```
Gereken VRAM ≈ Model_Boyut_GB × Quant_Carpani + Context_Overhead

Quant Carpanlari:
  FP16:   1.0x
  Q8_0:   0.55x
  Q4_K_M: 0.30x

Context Overhead ≈ (context_length / 1024) × 0.1 GB (yaklasik)

Ornek: 30B model, Q4_K_M, 32K context
  = 30 × 0.30 + (32768 / 1024) × 0.1
  = 9 + 3.2 = ~12.2 GB VRAM
```

### Hiz Optimizasyonu

| TEKNIK | ETKI | NASIL |
|--------|------|-------|
| Flash Attention 2 | 2-4x hiz | `--enable-flash-attn` (vLLM default) |
| KV Cache Quantization | %30 VRAM tasarruf | `--kv-cache-dtype fp8_e5m2` |
| Speculative Decoding | 2-3x hiz | Draft model + target model |
| Continuous Batching | Throughput artisi | vLLM default |
| Tensor Parallelism | Cok GPU dagitim | `--tensor-parallel-size N` |

---

## 9. KALITE KONTROL

### Asistan Kalitesini Degerlendirme

| KRITER | KONTROL | BASARILI |
|--------|---------|----------|
| Kod dogrulugu | Uretilen kod calisiyor mu? | Derlenir, test gecer |
| Bağlam anlama | Mevcut kodu dogru okuyor mu? | Yanlis dosya duzenleme yok |
| Tool kullanimi | Araclari dogru kullaniyor mu? | Gereksiz arac cagrisi yok |
| Talimat takibi | Kurallara uyuyor mu? | Format, stil tutarli |
| Guvenlik | Zafiyet uretiyor mu? | OWASP Top 10 temiz |
| Performans | Cevap suresi kabul edilebilir mi? | Agent modu < 60sn |

### A/B Test Yaklaşimi

```bash
# Ayni gorevi iki farkli konfigurasyonla test et:
# 1. Konfigürasyon A ile calistir, sonucu kaydet
# 2. Konfigürasyon B ile calistir, sonucu kaydet
# 3. Karsilastir: dogruluk, hiz, maliyet
```

---

## 10. ANTI-PATTERNS

| YAPMA | YAP |
|-------|-----|
| En buyuk modeli her is icin kullanma | Gorev-model eslesmesi yap |
| Raw model ciktisini dogrudan kullanma | Post-processing + validasyon ekle |
| Tek bir mega-prompt yazma | Katmanli prompt yapisi kullan |
| Context'i tamamen doldurma | RAG ile ilgili parcalari sec |
| Temperature'u 0 yapip yaraticilik bekleme | Gorev bazli ayarla |
| API key'i koda gomme | Ortam degiskeni kullan |
| Model ciktisina koru korune guvenme | Kritik islemlerde insan onayı al |
| GPU'yu tamamen doldurma | %85-90 utilization hedefle |

---

## 11. SORUN GIDERME

| SORUN | OLASI NEDEN | COZUM |
|-------|-------------|-------|
| Model cok yavas | VRAM yetersiz, swap kullanıyor | Daha kucuk quant veya model sec |
| Yanlis kod uretiyor | Dusuk kalite model veya yanlis prompt | Model yukselt, prompt iyilestir |
| Tool call yapamiyor | Model desteklemiyor | Tool-call destekli model sec |
| Context asimi | Cok buyuk dosya/cok dosya | max_tokens artir veya RAG kullan |
| Tekrarlayan cikti | frequency_penalty dusuk | frequency_penalty: 0.3-0.5 |
| Cevap ortada kesildi | max_tokens dusuk | max_tokens artir |
| OOM (out of memory) | VRAM yetersiz | Quant dusur, batch azalt |
| API baglanti hatasi | Port/URL yanlis | baseURL ve port kontrol et |
