# Continue Refactor Playbook

Bu dokuman, mevcut Continue config/rule yapisinda yapilan refactor'un ne oldugunu, neden yapildigini ve benzer mantigin baska bir agent sistemine nasil tasinacagini anlatir.

Amac: Benzer iyilestirmeleri daha sonra OpenCode tarafinda da uygulayabilmek icin net bir blueprint cikarmak.

---

## 1. Refactor'un Ana Hedefi

Eski yapida cok sayida fiziksel model entry vardi ve domain uzmanligi model bazinda tekrar tekrar yaziliyordu.

Yeni yapinin hedefi:

- fiziksel model sayisini azaltmak
- davranis katmanini netlestirmek
- domain uzmanligini rule katmanina tasimak
- config'i daha bakilabilir, olceklenebilir ve tune edilebilir hale getirmek
- model degistirme maliyetini dusurmek

Kisa ozet:

- daha az model
- daha guclu rule sistemi
- daha net davranis katmani
- daha profesyonel ve bakimi kolay mimari

---

## 2. Eski Problem Alanlari

Refactor oncesi ana sorunlar:

1. Ayni domain bilgisi farkli model ailelerinde tekrar ediyordu.
2. Model filosu kalabalik oldugu icin operasyonel sadelik dusuyordu.
3. Domain uzmanligi model secimiyle fazla bagliydi.
4. Prompt/rule/model sinirlari yeterince sade degildi.
5. Dokumantasyon config ile drift etmeye baslamisti.

Bu durum uzun vadede su riskleri olusturuyordu:

- tuning zorlasir
- model degisimi pahaliya mal olur
- davranis farklari kontrolsuz buyur
- ayni uzamanlik farkli yerlerde kopyalanir
- hangi katmanin sorumlulugu ne, bulaniklasir

---

## 3. Yeni Mimari

Yeni yapi, uzmanlik yerine davranis merkezli cekirdek motorlar kullaniyor.

### Cekirdek modeller

1. `Hard-Engineer`
   - ana kaliteli motor
   - derin analiz, kritik degisiklik, production odakli isler

2. `Soft-Engineer`
   - hizli motor
   - kucuk bug fix, kisa refactor, hizli analiz

3. `Vision-Engineer`
   - gorsel ve uzun-context motoru
   - sematik, tablo, image input, uzun baglam

4. `GLM-Hard-Engineer`
   - GLM tarafinda hard-engine davranisini koruyan alternatif motor
   - yedek gibi davranmasin; ayni kalite personasiyle calissin

5. `Rerank-Model`
   - dedicated reranker

6. `Embedding-Model`
   - embedding modeli

---

## 4. En Kritik Tasarim Karari

### Domain uzmanligi modelde degil, rules katmaninda olacak

Bu refactor'un merkez karari budur.

Uzmanlik artik su katmanlardan gelir:

- `alwaysApply` kurallari
- `globs` ile otomatik aktif olan domain kurallari
- gerekirse kullanicinin manuel ekledigi ilgili rule dosyalari

Yani:

- FPGA uzmani diye ayri model tanimlamak yerine
- aktif model + FPGA rules birlikte calisir

Bu sayede:

- model ile domain birbirinden ayrilir
- ayni domain mantigi farkli motorlara tasinabilir
- yeni model geldiginde sadece model katmani degisir
- domain kurallari korunur

---

## 5. Bilerek Kaldirilan Seyler

### Prompt katmani kaldirildi

Onceden slash prompt benzeri bir uzmanlik katmani dusunuldu, sonra bilincli olarak kaldirildi.

Gerekce:

- `alwaysApply + globs + manuel rule ekleme` zaten yeterli
- ek prompt katmani gereksiz zihinsel yuk olusturur
- uzmanligi fazla yere dagitmak sistem karmasasi yaratir

Son karar:

- uzmanlik promptlari yok
- uzmanlik rule sistemiyle gelir
- gerekirse kullanici `@rule` mantigi ile ilgili rule'u elle ekler

---

## 6. Bilerek Korunan Seyler

### Ayrik anchor yapisi korundu

Model anchor'lari tek bir ortak block'ta birlestirilmedi.

Sebep:

- gelecekte model bazli tuning ihtiyaci olabilir
- `top_p`, `presence_penalty`, `frequency_penalty`, `contextLength`, `maxTokens` ayrisabilir
- her model ailesi icin bagimsiz tuning esnekligi korunmak istendi

Korunan anchor mantigi:

- GLM anchor'lari ayri
- Kimi anchor'lari ayri
- Qwen hard/expert anchor'lari ayri
- Qwen fast anchor'lari ayri

Bu karar ozellikle operasyonel olarak dogru kabul edildi.

---

## 7. GLM Tarafinda Ozel Korunan Ayar

Tarihsel kontrol yapildi ve `GLM-4.7` icin su degerlerin bilincli olarak korundugu dogrulandi:

- `frequency_penalty: 0.2`
- `presence_penalty: 0.1`

Bu degerler rastgele secilmedi.

Gecmis commit analizi su sonucu verdi:

- ilk asamada `0.0 / 0.0` vardi
- sonra loop/incremental davranisi azaltmak icin `0.2 / 0.1` secildi
- migration sirasinda farkli degerlere kaydi
- sonrasinda bilincli bicimde tekrar `0.2 / 0.1`'e donuldu

Sonuc:

- GLM tarafindaki penalty ayari tarihsel olarak korunmasi gereken bir davranis ayaridir
- benzer refactor baska sistemde yapilirken bu tip tarihsel tuning bilgisi kaybedilmemelidir

---

## 8. Rules Sisteminin Yeni Rolu

Bu refactor'dan sonra rules katmani artik bir yan unsur degil, sistemin ana uzmanlik omurgasi haline geldi.

### Rules sorumluluklari

1. Genel davranis protokolu
2. Tool disiplini
3. Production kalite bari
4. Domain teknik uzmanligi
5. Dokumantasyon koruma davranisi
6. Git guvenligi ve is akisi

### Ilke

Rule setleri kirpilabilir ama bilgi kaybi olmamali.

Bu cok onemliydi:

- daha kisa olmak ugruna kalite dusmemeli
- profesyonellikten taviz verilmemeli
- davranis seviyesi Codex / Claude Code / Copilot ayarinda kalmali

Yani hedef sadece sade olmak degil, ayni zamanda ust seviye muhendislik davranisini korumaktir.

---

## 9. Rule Tarafinda Yapilan Ozel Iyilestirmeler

### Dokumantasyon rule'u mimari-nor hale getirildi

Eski dilde "dokumantasyon uzman modeli" gibi ifadeler vardi.
Bu ifadeler kaldirildi.

Yeni anlayis:

- aktif model calisir
- ilgili domain rule'lari uzmanligi saglar

### Yeni Git rule'u eklendi

`10-git.md` eklendi.

Bu rule su odaklari getirir:

- durum analizi once gelir
- destructive git adimlarinda risk/uyari zorunlu
- commit hijyeni korunur
- conflict ve history etkisi dusunulur
- ham komut ciktilari yerine profesyonel ozet sunulur

Bu onemli, cunku Git konusu dosya uzantisi ile otomatik cikmayan bir uzmanlik alanidir.

---

## 10. Dokumantasyon Hizasi

Config degisince README de hizalandi.

Yapilanlar:

- eski uzman model listesi kaldirildi
- yeni cekirdek motor mimarisi anlatildi
- sampling/context tablolari guncellendi
- GLM-4.7 ayarlari ve Qwen ayrimi netlestirildi
- yeni `10-git.md` rule'u dokumante edildi

Ilke:

- config degisirse README drift etmemeli
- mimari degisim mutlaka docs'a yansimali

---

## 11. Guvenlik ve Temizlik Prensibi

Push oncesi repo genelinde kurum/organizasyon adi kontrol edildi.

Prensip:

- org-ozel veri kalmamali
- gereksiz kurumsal isimler repo icinde gecmemeli
- config pushlaniyorsa hassas icerik bilerek degerlendirilmeli

Benzer refactor OpenCode tarafinda yapilirken de bu hijyen korunmali.

---

## 12. Dil Politikasi

Rule, policy, skill, prompt, instruction ve benzeri davranis icerikleri varsayilan olarak Turkce olmalidir.

Ilke:

- sistem davranisini belirleyen metinler Turkce yazilmali
- dokumantasyon ve aciklama metinleri Turkce olmali
- teknik terimler gerekliyse Ingilizce kalabilir
- dil tercihi mimari sadeleşme bahanesiyle kaybedilmemeli

Bu ozellikle onemlidir, cunku davranis katmani ne kadar kurallara tasinirse bu metinlerin dili de o kadar merkezi hale gelir.

---

## 13. OpenCode Tarafina Tasinacak Ana Dersler

Bu refactor birebir kopyalanmak zorunda degil, ama su prensipler tasinmali:

### A. Model sayisini azalt, rol sayisini netlestir

- domain bazli onlarca model yerine
- 3-4 cekirdek davranis modeli kullan

### B. Uzmanligi config/model yerine rule/policy katmanina tasi

- dosya tipi
- gorev tipi
- risk sinifi
- tool disiplini
- kalite bari

hepsi policy/rule katmaninda dursun

### C. Ayrik tuning bloklarini koru

Her model ailesinin ayarlari ayri kalmali.

Sebep:

- bugun ayni olan parametreler yarin ayrisabilir
- tuning esnekligi mimari sadeleşme ugruna feda edilmemeli

### D. Prompt katmanini sadece gercekten gerekiyorsa kullan

Eger rules zaten isi coziyorsa ekstra prompt katmani ekleme.

### E. Docs drift olmasin

Config degistiyse README / docs da guncellenmeli.

### F. Tarihsel tuning kararlarini kaybetme

Ozellikle sampling ve ceza parametreleri gibi alanlarda:

- hangi deger neden secildi
- hangi committe degisti
- neden geri donuldu

bilgisi korunmali.

---

## 14. Bu Refactor'un Ozet Ciktisi

Kisa ozetle yapilan degisiklik su:

- uzman-model filosundan
- rule-merkezli cekirdek motor mimarisine gecildi

Yani sistem artik soyle calisiyor:

- model davranisi cekirdek motorla belirlenir
- domain uzmanligi rules ile belirlenir
- kalite bari alwaysApply ile korunur
- manuel uzmanlik gerekiyorsa ilgili rule elle eklenir

Bu tasarim daha:

- sade
- bakimi kolay
- profesyonel
- tune edilebilir
- gelecekte baska sistemlere tasinabilir

---

## 15. OpenCode Icin Uygulama Talimati Olarak Kisa Versiyon

Eger bu dokuman baska bir agente verilecekse, en kisa operasyonel talimat su olabilir:

1. Fiziksel model sayisini azalt.
2. Cekirdek davranis motorlari tanimla: hard / soft / vision / alternatif-hard.
3. Domain uzmanligini rule/policy katmanina tasi.
4. Uzmanlik promptlarini kaldir; gerekiyorsa manuel rule ekleme mantigi kullan.
5. Model tuning anchor'larini birlestirme; ayri tut.
6. Tarihsel sampling kararlarini koru.
7. Docs'u yeni mimariye gore hizala.
8. Guvenlik ve kurumsal veri hijyenini push oncesi kontrol et.
9. Rule / skill / prompt / instruction iceriklerini Turkce yaz.

---

## 16. Bu Refactor'da Uygulanan Dosya Duzeyi Degisiklikler

- `config.yaml`
  - model filosu sadeleştirildi
  - prompt katmani kaldirildi
  - anchor yapisi korundu
  - GLM hard persona hizalandi

- `.continue/rules/06-documentation.md`
  - model-spesifik ifade mimari-nor hale getirildi

- `.continue/rules/10-git.md`
  - yeni git workflow rule'u eklendi

- `README.md`
  - yeni mimariye gore tamamen hizalandi

---

Bu dokumanin amaci "ne degisti" kadar "neden boyle degisti" sorusuna da cevap vermektir. OpenCode tarafinda benzer bir refactor yapilacaksa, ayni mantik korunmali: sadelik ugruna uzmanlik kaybedilmemeli, profesyonellik ugruna da gereksiz karmasa uretilmemelidir.
