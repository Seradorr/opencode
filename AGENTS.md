# OpenCode AI Asistan Konfigürasyon Projesi

Bu proje, OpenCode AI kodlama asistanının Copilot/Claude Code seviyesinde çalışmasını sağlayan profesyonel konfigürasyon deposudur.

## Proje Yapısı

- `opencode.jsonc` — Ana konfigürasyon (modeller, agentlar, izinler, formatter, LSP)
- `.opencode/core.md` — Tüm agentlar için geçerli temel kurallar (Core Protocol)
- `.opencode/agents/` — Agent prompt dosyaları (build, plan, review, refactor, docs, test, fast)
- `.opencode/skills/` — Domain-spesifik bilgi paketleri (fpga, vitis, dotnet, python, document, ai-toolkit)
- `docs/` — Ek dokümantasyon ve rehber dosyaları

## Teknoloji Alanları

Bu konfigürasyon aşağıdaki teknoloji alanlarını kapsar:

- **FPGA**: VHDL, Verilog, SystemVerilog, Vivado, XDC constraints
- **Embedded**: Xilinx Vitis, bare-metal C/C++, BSP, DMA, interrupt
- **C#/.NET**: WPF/MVVM, EF Core, Minimal API, async/await
- **Python**: Scripting, otomasyon, pytest, asyncio, CLI tools
- **Dokümantasyon**: Markdown, README, teknik spesifikasyon, ADR

## Agent Mimarisi

| Agent | Mod | Model | Görev |
|-------|-----|-------|-------|
| build | primary | Qwen3.5-397B | Ana geliştirme, kod yazma, debug |
| plan | primary | Qwen3.5-397B | Analiz, planlama, mimari (read-only tercih) |
| fast | primary | Qwen3.5-35B | Hızlı görevler, günlük kodlama |
| review | subagent | Qwen3.5-397B | Kod inceleme (read-only) |
| refactor | subagent | Qwen3.5-397B | Optimizasyon, modernizasyon |
| docs | subagent | Kimi-K2.5 | Teknik dokümantasyon |
| test | subagent | GLM-4.7-FP8 | Test yazma, testbench |

### Kullanım

- **Tab** ile primary agentlar arası geçiş: build ↔ plan ↔ fast
- **@review**, **@refactor**, **@docs**, **@test** ile subagent çağrısı
- Plan modu dosya değiştirmeden önce onay ister

## Kodlama Standartları

- Açıklamalar Türkçe, teknik terimler İngilizce
- Placeholder (`...`, `// existing code`) YASAK
- Dosya okumadan düzenleme YASAK
- Her değişiklik tam ve çalıştırılabilir olmalı
- Güvenlik: secret/key/credential kopyalama/yayma yasak

## Konfigürasyon Kuralları

- `opencode.jsonc` düzenlerken JSON schema'ya uygunluk kontrol edilmeli
- Skill dosyaları `.opencode/skills/<lowercase-name>/SKILL.md` formatında olmalı
- `name` alanı dizin adıyla birebir eşleşmeli: `^[a-z0-9]+(-[a-z0-9]+)*$`
- Agent prompt dosyaları `.opencode/agents/<name>.md` formatında
- Yeni skill eklerken mevcut format ve yapıya uyulmalı

## Kritik Dosyalar

Bu dosyalarda değişiklik yapmadan önce ONAY iste:

| Alan | Dosyalar |
|------|----------|
| FPGA | `*.xdc`, top-level entity, clock/reset modülleri |
| Embedded | `lscript.ld`, FSBL, ISR, `Makefile` |
| .NET | `*.csproj`, `Program.cs`, `appsettings.json` |
| Python | `pyproject.toml`, `setup.py`, `__init__.py` |
| Genel | `.gitignore`, CI/CD config, `opencode.jsonc` |

## Build/Test Komutları

| Alan | Komut |
|------|-------|
| FPGA | Vivado sentez/implementasyon, xsim simülasyon |
| Embedded C | `make`, `xsdb` debug, Vitis IDE build |
| C#/.NET | `dotnet build`, `dotnet test` |
| Python | `pytest`, `python -m black`, `mypy` |
