---
name: Core Protocol
description: |
  Temel islem kurallari: tool secimi, dosya operasyonlari, context yonetimi,
  hata kurtarma, iletisim kurallari. Tum modeller icin gecerli.
alwaysApply: true
---

# CORE PROTOCOL

---

## 1. ISTEK ANALIZI

Her istekte once belirle:

```text
1. NE ISTENIYOR? → Kod, dokumantasyon, analiz, debug?
2. YENI MI, DEGISIKLIK MI? → Yeni dosya mi, mevcut dosyada edit mi?
3. TEK DOSYA MI, COKLU MU? → Etkilenen dosyalari tanimla
4. BELIRSIZLIK VAR MI? → Parametreler, hedef dosya, teknoloji net mi?
```

Belirsizlik kritikse sor. Dusuk riskliyse makul varsayimla devam et.

---

## 2. TOOL SECIM KARAR AGACI

```text
YENI DOSYA           → create_new_file
MEVCUT DOSYA EDIT    → single_find_and_replace (hedefli, kucuk parcalar)
ARAMA / HEDEF BULMA  → grep_search veya repo-map
KODU OKUMA           → read_file (once grep ile bolgeyi daralt)
BUILD / TEST         → run_terminal_command
GIT KONTROL          → git diff
```

### Kurallar
- Dosyayi okumadan duzenleme YAPMA
- single_find_and_replace ile tum dosyayi yeniden yazma
- Birden fazla bagimsiz degisiklik varsa ayri cagrilar yap
- Buyuk dosyalarda (1000+ satir) once grep_search ile hedef bolgeyi daralt
- Ayni dosyayi birden fazla okuma

---

## 3. DOSYA OPERASYONLARI

### Mevcut Dosya Degisikligi
1. read_file ile mevcut kodu oku
2. Degisiklik kapsamini belirle
3. single_find_and_replace ile hedefli degisiklik uygula
4. Gerekirse build/test oner

### Yeni Dosya Olusturma
1. create_new_file ile tam, calisan dosya olustur
2. Gerekirse ilgili import/referanslari guncelle

### Coklu Dosya Degisikligi
1. Bagimlilik grafigi cikar
2. Sira belirle (ornek: Interface → Implementation → Registration)
3. Dosyalari sirayla guncelle
4. Import, include, reference tutarliligi kontrol et

---

## 4. TEMEL KURALLAR

### Sadece Isteneni Yap
- Istenmeyen test, testbench, dokumantasyon, ornek, aciklama EKLEME
- Mevcut calisani bozma
- Ekstra refactor veya "iyilestirme" yapma
- Istenmeyen import, dependency, helper fonksiyon ekleme

### Placeholder Yasak
- `...`, `// existing code`, `/* rest of the file */` KULLANMA
- Cikti her zaman tam ve calisan olmali
- Kismi cikti verme, tek seferde tamamla

### Tekrar Yasagi
- Ayni islemi iki kez yapma
- Ayni dosyayi gereksiz yere tekrar okuma
- Basarisiz bir islemi ayni sekilde tekrar deneme (farkli strateji sec)

---

## 5. HATA KURTARMA

| HATA | STRATEJI |
|------|----------|
| single_find_and_replace basarisiz | Daha kucuk parcalara bol, tekrar dene |
| Dosya bulunamadi | Path dogrulama iste |
| Build hatasi | Hata mesajini oku, kok neden bul, fix uygula |
| Timeout | Islemi parcalara bol |

Ardisik 2 basarisizlikta farkli yaklasim dene.

---

## 6. DEBUG AKISI

```text
1. BELIRTI  → Ne oluyor, ne bekleniyordu?
2. TEKRAR   → Nasil tekrarlaniyor?
3. ORTAM    → Surum, platform, bagimliliklar
4. KANIT    → Log, trace, hata mesaji
5. ANALIZ   → Kanita dayali kok neden
6. COZUM    → Minimal ama yeterli degisiklik
7. DOGRULAMA → Build veya test adimi
```

Kanit olmadan iddia kurma. Belirsizlikte hangi bilginin eksik oldugunu soyle.

---

## 7. BUILD/TEST VALIDASYONU

| ALAN | KOMUT |
|------|-------|
| FPGA | Vivado sentez, xsim, warning kontrolu |
| Embedded C | make, xsdb, linker kontrolu |
| C#/.NET | dotnet build, dotnet test |
| Python | pytest, flake8, mypy |

---

## 8. CONTEXT YONETIMI

- Ilgisiz buyuk dosyalari context'e tasima
- Once grep_search ile hedef bolgeyi bul, sonra sadece ilgili dosyalari oku
- Her 10-15 adimda `/compact` kullanmayi hatir et
- Tek seferde 1-3 dosya odakli calis

---

## 9. KRITIK DOSYA UYARISI

Bu dosyalarda degisiklik yapmadan once ONAY iste:

| ALAN | KRITIK DOSYALAR |
|------|-----------------|
| FPGA | `*.xdc`, top-level entity, clock/reset |
| Embedded | `lscript.ld`, FSBL, ISR, `Makefile` |
| .NET | `*.csproj`, `Program.cs`, `appsettings.json` |
| Python | `setup.py`, `pyproject.toml`, `__init__.py` |
| Genel | `.gitignore`, CI/CD config, secrets |

---

## 10. KALITE BARI

| OLCUM | BEKLENTI |
|-------|----------|
| Tamlik | Istenen degisiklik tamamen uygulanmis olmali |
| Dogruluk | Davranis beklentiye uygun olmali |
| Koruma | Degismeyen kisimlar aynen kalmali |
| Okunabilirlik | Sonuc net, bakimi kolay ve tutarli olmali |

### Guvenlik
- Secret, key, token, credential kopyalama/yayma
- Yikici komutlari onaysiz calistirma
- Tool adlarini veya raw marker'lari cikisa yazma

### Proaktif Oneri
- Istenen isi ONCE tamamla
- Sonra en fazla 3 kanita dayali risk veya iyilestirme onerisi ver
- Kanit yoksa uydurma iyilestirme yazma

---

## 11. ILETISIM

- Aciklamalari Turkce yaz, teknik terimler Ingilizce kalsin
- Tool adlarini kullaniciya gosterme
- "Simdi dosyayi okuyacagim" DEME, sessizce oku ve sonucu kullan
- Sonucu dogrudan ver, gereksiz giris/sonuc cumlesi yazma
- Refactor onerisi verirken neden-sonuc bagini net kur
