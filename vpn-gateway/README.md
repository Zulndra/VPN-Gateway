# VPN Gateway dengan WireGuard  
Dokumentasi SEAMEO SEAMOLEC

## 1. Deskripsi
Proyek ini membuat **VPN Gateway terpusat** untuk menghubungkan:
- **Server Client (remote)** – Server yang berada di luar jaringan kantor.
- **VPN Gateway (Server WireGuard)** – Menjadi pusat jalur VPN.
- **Router Kantor (MikroTik)** – Router kantor yang menyediakan akses LAN.

Tujuan: Menghubungkan server remote ke jaringan lokal kantor (192.168.10.0/24) dengan koneksi terenkripsi.

## 2. Topologi
```
Server Client  <---->  VPN Gateway (WireGuard)  <---->  MikroTik Router  <---->  LAN Kantor
```

Contoh IP:
| Perangkat              | IP Publik / Lokal                    | Keterangan        |
|------------------------|--------------------------------------|-------------------|
| Server Client          | –                                    | Terkoneksi ke VPN |
| VPN Gateway (Ubuntu)   | Publik: x.x.x.x <br> WG: 10.8.0.1/24 | Pusat VPN         |
| MikroTik Router        | LAN: 192.168.10.1                    | Gateway LAN       |
| Jaringan LAN Kantor    | 192.168.10.0/24                      | Tujuan akses      |

---

## 3. Persiapan
- Server VPN (Ubuntu 20.04 atau lebih baru).
- Server Client (Ubuntu/Debian, dsb).
- Router MikroTik (RouterOS v7+ yang sudah mendukung WireGuard).
- Akses root/administrator ke semua perangkat.

---

## 4. Instalasi & Konfigurasi

### 4.1 Instal WireGuard di VPN Gateway (Ubuntu)
```bash
sudo apt update && sudo apt install -y wireguard
```

#### Generate key pair
```bash
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key
```

#### Konfigurasi `/etc/wireguard/wg0.conf`
```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <ISI server_private.key>

# Routing ke LAN kantor
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

> Ganti `eth0` dengan nama interface internet server.

Aktifkan:
```bash
sudo systemctl enable --now wg-quick@wg0
```

---

### 4.2 Tambahkan Peer (Server Client)
Pada Gateway, edit `wg0.conf`:
```ini
[Peer]
PublicKey = <PUBLIC_KEY_CLIENT>
AllowedIPs = 10.8.0.2/32
```
Restart:
```bash
sudo systemctl restart wg-quick@wg0
```

---

### 4.3 Konfigurasi Server Client
Di server client:
```bash
sudo apt update && sudo apt install -y wireguard
wg genkey | tee client_private.key | wg pubkey > client_public.key
```
`/etc/wireguard/wg0.conf`
```ini
[Interface]
Address = 10.8.0.2/24
PrivateKey = <client_private.key>
DNS = 8.8.8.8

[Peer]
PublicKey = <server_public.key>
Endpoint = <IP_PUBLIK_GATEWAY>:51820
AllowedIPs = 192.168.10.0/24, 10.8.0.0/24
PersistentKeepalive = 25
```
Aktifkan:
```bash
sudo systemctl enable --now wg-quick@wg0
```

---

## 5. Konfigurasi MikroTik Router (RouterOS v7)

Masuk Winbox/CLI RouterOS.

### 5.1 Buat Interface WireGuard
```
/interface wireguard add name=wg-seamolec listen-port=51820 private-key="<PRIVATE_KEY_ROUTER>"
```

### 5.2 Buat Peer ke VPN Gateway
```
/interface wireguard peers add interface=wg-seamolec public-key="<PUBLIC_KEY_GATEWAY>" endpoint="<IP_PUBLIK_GATEWAY>:51820" allowed-address=10.8.0.0/24
```

### 5.3 Tambahkan IP ke Interface
```
/ip address add address=10.8.0.3/24 interface=wg-seamolec
```

### 5.4 Routing ke LAN Kantor
Jika LAN kantor: `192.168.10.0/24`
```
/ip route add dst-address=192.168.10.0/24 gateway=10.8.0.1
```

> Sesuaikan IP `10.8.0.3` sebagai IP WireGuard di router.

---

## 6. Verifikasi
- **Cek status WireGuard (Ubuntu):**
  ```bash
  sudo wg show
  ```
- **Ping dari Server Client ke LAN kantor:**
  ```bash
  ping 192.168.10.1
  ```
- Pastikan paket sudah melewati tunnel.

---

## 7. Tips & Catatan
- Pastikan port 51820/UDP terbuka di firewall VPS / server.
- Gunakan kunci yang aman dan backup file private key.
- Gunakan `PersistentKeepalive = 25` pada client untuk koneksi stabil di NAT.

---

## 8. Hasil
Dengan konfigurasi ini:
- Server remote dapat mengakses jaringan kantor secara aman.
- VPN Gateway berperan sebagai jalur terpusat sehingga manajemen koneksi lebih mudah.

---

## Lisensi
Dokumentasi ini dibuat di **SEAMEO SEAMOLEC**, dapat disesuaikan sesuai kebutuhan.
