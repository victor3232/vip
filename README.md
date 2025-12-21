

### INSTALL SCRIPT 
- MAINTENANCE

### TESTED ON OS 
- UBUNTU 20.04.05
- DEBIAN 10

### SIMPLE FEATURE
- LIMIT IP
- LIMIT QUOATA XRAY
- BOT NOTIF MULLOG & QUOTA HABIS
- SIMPLE CREATE WITH TELEGRAM BOT
- SUPPORT SEMUA METHOD INJECT


### PORT INFO
```
- MULTI PORT 443 / 80
```
### ADD ON
```
function install_xray() {
    clear
    print_install "Core Xray v1.7.5 (Stable Version)"
    
    # 1. Buat Direktori
    mkdir -p /etc/xray
    mkdir -p /var/log/xray
    mkdir -p /usr/local/share/xray
    chown www-data.www-data /var/log/xray
    chmod +x /var/log/xray
    touch /var/log/xray/access.log
    touch /var/log/xray/error.log
    
    # 2. Download Xray Core v1.7.5 (Versi Paling Stabil untuk Config Lama)
    # Kita pakai link official GitHub tapi versi 1.7.5
    ARCH=$(uname -m)
    if [[ $ARCH == "x86_64" ]]; then
        # Versi 1.7.5 Official (Stabil)
        LINK="https://github.com/XTLS/Xray-core/releases/download/v1.7.5/Xray-linux-64.zip"
    elif [[ $ARCH == "aarch64" ]]; then
        LINK="https://github.com/XTLS/Xray-core/releases/download/v1.7.5/Xray-linux-arm64-v8a.zip"
    else
        print_error "Arsitektur tidak didukung!"
        exit 1
    fi

    echo "Downloading Xray Core v1.7.5..."
    wget -q -O /tmp/xray.zip "$LINK"
    
    # 3. Cek apakah download sukses
    if [ ! -s /tmp/xray.zip ]; then
        echo "Download Gagal! Mencoba Link Backup..."
        # Link Backup jika GitHub limit (Opsional, arahkan ke repo Anda jika punya backup)
        wget -q -O /tmp/xray.zip "https://github.com/dhezpiere/Xray-core/releases/download/v1.7.5/Xray-linux-64.zip"
    fi

    # 4. Unzip dan Pasang
    unzip -o /tmp/xray.zip -d /tmp/xray_bin > /dev/null 2>&1
    mv /tmp/xray_bin/xray /usr/local/bin/xray
    chmod +x /usr/local/bin/xray
    rm -rf /tmp/xray.zip
    rm -rf /tmp/xray_bin
    
    # 5. Ambil Config & Service (Sama seperti sebelumnya)
    wget -O /etc/xray/config.json "${REPO}limit/config.json" >/dev/null 2>&1
    wget -O /etc/systemd/system/runn.service "${REPO}limit/runn.service" >/dev/null 2>&1

    # 6. Download GeoData
    wget -q -O /usr/local/share/xray/geosite.dat "https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat"
    wget -q -O /usr/local/share/xray/geoip.dat "https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat"

    domain=$(cat /etc/xray/domain)
    
    # Settings UP Nginx Server & HAProxy
    clear
    curl -s ipinfo.io/city >>/etc/xray/city
    curl -s ipinfo.io/org | cut -d " " -f 2-10 >>/etc/xray/isp
    print_install "Memasang Konfigurasi Packet"
    wget -O /etc/haproxy/haproxy.cfg "${REPO}limit/haproxy.cfg" >/dev/null 2>&1
    wget -O /etc/nginx/conf.d/xray.conf "${REPO}limit/xray.conf" >/dev/null 2>&1
    sed -i "s/xxx/${domain}/g" /etc/haproxy/haproxy.cfg
    sed -i "s/xxx/${domain}/g" /etc/nginx/conf.d/xray.conf
    
    curl ${REPO}limit/nginx.conf > /etc/nginx/nginx.conf
    cat /etc/xray/xray.crt /etc/xray/xray.key | tee /etc/haproxy/hap.pem

    chmod +x /etc/systemd/system/runn.service

    # Create Service Xray (Standard)
    rm -rf /etc/systemd/system/xray.service.d
    cat >/etc/systemd/system/xray.service <<EOF
[Unit]
Description=Xray Service
Documentation=https://github.com/XTLS/Xray-core
After=network.target nss-lookup.target

[Service]
User=www-data
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/xray run -config /etc/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
EOF

    print_success "Xray Core v1.7.5 Installed"
}
```

### TAMPILAN SCRIPT
![IMG_20231217_190421](https://github.com/victor3232/vip/assets/44395332/2651a342-67a3-4d01-b52b-4995abc7afac)

