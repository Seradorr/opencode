# MCP (Model Context Protocol) Kurulum ve Kullanım Rehberi

## MCP Nedir?

MCP, AI asistanının dış sistemlere erişmesini sağlayan açık bir protokoldür.
AI modeli tek başına dosya okuyamaz, veritabanı sorgulayamaz, terminal çalıştıramaz.
MCP server'lar bu köprüyü kurar.

```
[OpenCode/AI] ←→ [MCP Client (OpenCode içinde)] ←→ [MCP Server] ←→ [Dış Sistem]
```

**Basit anlatım:** MCP server = AI'a verdiğin araç kutusu.
- `server-filesystem` → AI'a dosya okuma/yazma/arama yeteneği verir
- `server-memory` → AI'a konuşmalar arası kalıcı hafıza verir

## @modelcontextprotocol/server-* Paketleri

Bunlar npm paketleridir (Node.js modülleri). Her biri bir MCP server çalıştırır:

| PAKET | NE YAPAR | AI NE KAZANİR |
|-------|----------|----------------|
| `server-filesystem` | Belirtilen klasörlerde dosya okur/yazar/arar | Proje dosyalarına erişim |
| `server-memory` | JSON tabanlı kalıcı hafıza | Session'lar arası bilgi hatırlama |
| `server-sqlite` | SQLite veritabanı sorguları | Veritabanı analizi |
| `server-fetch` | HTTP istekleri gönderme | İntranet API'lere erişim |
| `server-git` | Git işlemleri (log, diff, blame) | Repo geçmişi analizi |

## İntranet Ortamında Kurulum (Adım Adım)

İnternet olmayan makinede `npx -y` ÇALIŞMAZ. Paketleri önceden kurman gerekir.

### Yöntem 1: İnterneti Olan Makinede Hazırlık

```powershell
# 1. İnterneti olan bir makinede boş klasör oluştur
mkdir C:\mcp-paketler
cd C:\mcp-paketler

# 2. İstediğin paketleri indir
npm pack @modelcontextprotocol/server-filesystem
npm pack @modelcontextprotocol/server-memory

# 3. .tgz dosyaları oluşur:
#    modelcontextprotocol-server-filesystem-x.x.x.tgz
#    modelcontextprotocol-server-memory-x.x.x.tgz

# 4. Bu dosyaları hedef makineye taşı (USB/ağ paylaşımı vb.)
```

### Yöntem 2: Offline Makinede Kurulum

```powershell
# 1. OpenCode proje klasörüne git
cd C:\Users\alise\Desktop\PROJELER\VS\OpenCode

# 2. .tgz dosyalarını buraya kopyala, sonra kur:
npm install ./modelcontextprotocol-server-filesystem-x.x.x.tgz
npm install ./modelcontextprotocol-server-memory-x.x.x.tgz

# 3. Kurulum sonrası node_modules/ altına yerleşir:
#    node_modules/@modelcontextprotocol/server-filesystem/
#    node_modules/@modelcontextprotocol/server-memory/

# 4. .tgz dosyalarını silebilirsin
```

### Yöntem 3: Doğrudan npm install (eğer npm registry erişimi varsa)

```powershell
cd C:\Users\alise\Desktop\PROJELER\VS\OpenCode
npm install @modelcontextprotocol/server-filesystem
npm install @modelcontextprotocol/server-memory
```

## opencode.jsonc Konfigürasyonu

Paketler `node_modules/` altına kurulduktan sonra `opencode.jsonc`'de şu şekilde tanımla:

```jsonc
"mcp": {
  "filesystem": {
    "type": "local",
    "command": ["node",
      "node_modules/@modelcontextprotocol/server-filesystem/dist/index.js",
      "C:/Users/alise/Desktop/PROJELER"],
    "enabled": true
  },
  "memory": {
    "type": "local",
    "command": ["node",
      "node_modules/@modelcontextprotocol/server-memory/dist/index.js"],
    "enabled": true
  }
}
```

**Açıklama:**
- `type: "local"` → Yerel process olarak çalışır
- `command` → Node.js ile doğrudan paketteki JS dosyasını çalıştırır (npx gerektirmez)
- `"C:/Users/alise/Desktop/PROJELER"` → filesystem server'ın erişebileceği kök klasör
- `enabled: true` → Aktif

## Pratik Kullanım Senaryoları

### filesystem server ile
AI asistanına chat'te şunları sorabilirsin:
- "PROJELER klasöründeki tüm .vhd dosyalarını listele"
- "X projesindeki main.py dosyasını oku"
- "Bu config dosyasını güncelle"

### memory server ile
AI asistanı konuşmalar arası bilgi hatırlar:
- "Bu proje BRAM kullanıyor, bunu hatırla"
- "Daha önce timing closure hakkında ne söylemiştim?"

## MCP Olmadan da Çalışır mı?

EVET. OpenCode'un kendi yerleşik araçları zaten dosya okuma/yazma/terminal yapabilir.
MCP ek yetenek katmanıdır, zorunlu değildir. Eğer kurulum zahmetliyse
MCP bölümünü `opencode.jsonc`'den silebilirsin — hiçbir şey bozulmaz.

## Kurulu mu Kontrol Et

```powershell
# Node.js kurulu mu?
node --version

# Paketler kurulu mu?
Test-Path "node_modules/@modelcontextprotocol/server-filesystem"
Test-Path "node_modules/@modelcontextprotocol/server-memory"

# Manuel test
node node_modules/@modelcontextprotocol/server-filesystem/dist/index.js C:/test
# JSON çıktısı görürsen çalışıyordur (Ctrl+C ile kapat)
```
