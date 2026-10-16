# ufw-lab

# UFW Lab: Kural Sıralaması ve Kapatma

UFW (Uncomplicated Firewall) kurallarının nasıl sıralandığını ve bu sıralamanın neden önemli olduğunu adım adım gösteren küçük bir lab çalışması.

- **Sistem:** Kali Linux
- **Tarih:** 16 Ekim 2026
- **Konu:** UFW kural setleri, kural sıralaması, UFW'yi kapatma

## Amaç

UFW kuralları yukarıdan aşağıya değerlendirilir ve **ilk eşleşen kural uygulanır**. Bu lab'da aynı IP için bir `deny` kuralı önce `allow` kurallarından sonra (yanlış sıra), sonra en üste (doğru sıra) eklenerek farkı gösterilir.

Örnek IP olarak dokümantasyon için ayrılmış `203.0.113.5` (TEST-NET-3) kullanıldı. Bu adres gerçek bir sisteme ait değildir.

## Gereksinimler

- Linux (Kali, Debian, Ubuntu vb.)
- `sudo` yetkisi
- UFW kurulu olmalı:

```bash
sudo apt update
sudo apt install ufw -y
```

> **Uyarı:** Bu komutları sadece kendi makinende veya yetkin olduğun bir lab ortamında çalıştır. SSH ile bağlıysan `22/tcp` kuralını eklemeden UFW'yi açma, yoksa bağlantın kopar. `ufw reset` mevcut tüm kuralları siler (yedeği `/etc/ufw/` altına alır).

## Adım 1: Temiz başlangıç

```bash
sudo ufw disable
sudo ufw reset
```

Çıktı:
```
Firewall stopped and disabled on system startup
Resetting all rules to installed defaults. Proceed with operation (y|n)? y
Backing up 'user.rules' to '/etc/ufw/user.rules.20261016_144736'
...
```

Eski kurallar yedeklenip silindi.

## Adım 2: Varsayılan politikalar

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Çıktı:
```
Default incoming policy changed to 'deny'
Default outgoing policy changed to 'allow'
```

## Adım 3: İzin kuralları

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
```

Çıktı: `Rules updated` ve `Rules updated (v6)` (her iki kural için).

## Adım 4: Deny kuralı (yanlış sıra)

```bash
sudo ufw deny from 203.0.113.5
sudo ufw enable
sudo ufw status numbered
```

Çıktı:
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] Anywhere                   DENY IN     203.0.113.5
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
```

**Yorum:** Deny kuralı `[3]`'te, allow kurallarından sonra. Paket önce `22/tcp ALLOW` ve `80/tcp ALLOW` kurallarına eşleştiği için `203.0.113.5` yine 22 ve 80 portlarına erişebilir. Deny kuralı etkisiz kalır.

## Adım 5: Sırayı düzeltme

```bash
sudo ufw delete deny from 203.0.113.5
sudo ufw insert 1 deny from 203.0.113.5
sudo ufw status numbered
```

Çıktı:
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] Anywhere                   DENY IN     203.0.113.5
[ 2] 22/tcp                     ALLOW IN    Anywhere
[ 3] 80/tcp                     ALLOW IN    Anywhere
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
```

**Yorum:** Deny kuralı artık `[1]`'de. `203.0.113.5` ilk kurala takılır ve engellenir. Diğer herkes için 22 ve 80 izinli kalır.

## Adım 6: UFW'yi kapatma

```bash
sudo ufw disable
sudo ufw status
```

Çıktı:
```
Firewall stopped and disabled on system startup
Status: inactive
```

## Sonuç

- UFW kuralları yukarıdan aşağıya okunur, ilk eşleşen kural uygulanır.
- Özel (dar) kurallar, genel (geniş) kurallardan önce gelmelidir.
- Bir kuralı en üste almak için `ufw insert 1 ...` kullanılır.
- `ufw disable` güvenlik duvarını durdurur ve açılışta başlamasını engeller. Kurallar silinmez, silmek için `ufw reset` kullanılır.
- Kural dosyaları `/etc/ufw/` altındadır: `user.rules`, `before.rules`, `after.rules` ve IPv6 karşılıkları.

## Faydalı komutlar

| Komut | Ne yapar |
|-------|----------|
| `sudo ufw status verbose` | Durum ve varsayılan politikaları gösterir |
| `sudo ufw status numbered` | Kuralları numaralı listeler |
| `sudo ufw insert N <kural>` | Kuralı N. sıraya ekler |
| `sudo ufw delete N` | N numaralı kuralı siler |
| `sudo ufw enable` / `disable` | Güvenlik duvarını açar / kapatır |
| `sudo ufw reset` | Tüm kuralları siler |
