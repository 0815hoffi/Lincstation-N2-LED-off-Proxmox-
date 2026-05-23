# Lincstation-N2-LED-off-Proxmox-
Lincstation N2 LED off Proxmox Service Script


LED disable at Boot:
Connect over SSH to Proxmox Server:
_____________________________

1:

apt install i2c-tools -y

2:

cat > /usr/local/bin/lincled.sh << 'EOF'
#!/bin/bash

find_led_bus() {
    if i2cdetect -y 2 2>/dev/null | grep -q "26"; then
        echo "2"
        return
    fi
    for bus in {0..14}; do
        if i2cdetect -y $bus 2>/dev/null | grep -q "26"; then
            echo "$bus"
            return
        fi
    done
    echo "NO_BUS"
}

BUS=$(find_led_bus)
if [ "$BUS" = "NO_BUS" ]; then
    echo "ERROR: LED chip not found. Run 'i2cdetect -l'"
    exit 1
fi

echo "LED chip found on i2c-$BUS"

modprobe i2c-dev 2>/dev/null
sleep 1

i2cset -y $BUS 0x26 0x52 0x00
i2cset -y $BUS 0x26 0x54 0x00
i2cset -y $BUS 0x26 0x56 0x00
i2cset -y $BUS 0x26 0x58 0x00
i2cset -y $BUS 0x26 0x5A 0x00
i2cset -y $BUS 0x26 0x5C 0x00
i2cset -y $BUS 0x26 0x5E 0x00
i2cset -y $BUS 0x26 0x56 0x00
i2cset -y $BUS 0x26 0XB0 0x40
i2cset -y $BUS 0x26 0XB0 0x80

i2cset -y $BUS 0x26 0xB0 0x04 # white off
i2cset -y $BUS 0x26 0xB0 0x08 # red off 
i2cset -y $BUS 0x26 0x52 0x00 # blinking off

i2cset -y $BUS 0x26 0xB0 0x10 # white off
i2cset -y $BUS 0x26 0xB0 0x20 # red off 
i2cset -y $BUS 0x26 0x54 0x00 # blinking off

i2cset -y $BUS 0x26 0xB1 0x01 # white off
i2cset -y $BUS 0x26 0xB1 0x02 # red off 
i2cset -y $BUS 0x26 0x58 0x00 # blinking off


i2cset -y $BUS 0x26 0xB1 0x04 # white off
i2cset -y $BUS 0x26 0xB1 0x08 # red off
i2cset -y $BUS 0x26 0x5A 0x00 # blinking off

i2cset -y $BUS 0x26 0xB1 0x10 # white off
i2cset -y $BUS 0x26 0xB1 0x20 # red off
i2cset -y $BUS 0x26 0x5C 0x00 # blinking off

i2cset -y $BUS 0x26 0xB1 0x40 # white off
i2cset -y $BUS 0x26 0xB1 0x80 # red off
i2cset -y $BUS 0x26 0x5E 0x00 # blinking off

echo "SUCCESS: All Leds OFF (bus $BUS)"
EOF

3:
chmod +x /usr/local/bin/lincled.sh

5:
cat > /etc/systemd/system/lled.service << 'EOF'
[Unit]
Description=LincLEC
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/usr/local/bin/lincled.sh

[Install]
WantedBy=multi-user.target
EOF

6:
systemctl daemon-reload
systemctl start lled
systemctl status lled
systemctl enable lled
systemctl status lled
