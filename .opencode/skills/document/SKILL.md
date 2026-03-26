---
name: document
description: |
  Markdown, README, CHANGELOG ve teknik dokumantasyon dosyalarinin
  koruma, guncelleme ve format kurallari icin kullan.
---

# DOKUMANTASYON KURALLARI

Bu kurallar tum dokumantasyon dosyalari icin gecerlidir.
Dokumantasyon uzman modeli agent modunda calisir ve dosyalari dogrudan duzenleyebilir.

---

## 1. ICERIK KORUMA

Temel ilke: Kullanici acikca istemedikce mevcut icerigi silme.

| ISTEK | EYLEM |
|------|-------|
| "Ekle" | Mevcut + yeni = sonuc |
| "Guncelle" | Sadece belirtilen kisim degisir |
| "Sil" / "Kaldir" | Sadece belirtilen kisim silinir |
| "Yeniden yaz" | Belirtilen bolum tamamen degisir |

Belirsiz taleplerde silme yapma.

### Guncelleme Akisi

```text
1. Dosyanin tamamini oku
2. Mevcut yapiyi koru
3. Istenen degisikligi hedefli uygula
4. Geri kalan icerigi bozma
```

---

## 2. CHANGELOG KURALI

- Yeni girisler en uste eklenir
- Eski girisler korunur
- Mevcut format neyse ona uyulur
- Tarih formati ISO olmalidir: `YYYY-MM-DD`

---

## 3. YAPI KORUMA

Korunmasi gerekenler:
- Baslik hiyerarsisi
- Liste yapisi
- Tablolar
- Kod bloklari
- Linkler
- Mantikli bos satirlar

Yasak:
- Listeyi paragrafa cevirmek
- Tabloyu bozmak
- Baslik seviyesini gereksiz degistirmek

---

## 4. ASLA YAPILMAYACAKLAR

| YASAK | ACIKLAMA |
|------|----------|
| "Temizleme" bahanesiyle silme | Icerik kaybi dogurur |
| "Sadelestirme" diye keyfi kisaltma | Bilgi kaybi yaratir |
| Format bozma | Mevcut yapinin tutarliligini kirar |
| Placeholder birakma | Eksik ve kullanisiz cikti uretir |

---

## 5. PROAKTIF DOKUMAN KONTROLU

- Istenen guncelleme bittikten sonra, varsa en fazla 3 somut dokuman iyilestirme onerisi sun
- Oneriler mevcut yapinin eksikligi veya belirsizligi ile iliskili olmali
- Uydurma eksik listesi olusturma
