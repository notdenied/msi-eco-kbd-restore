# MSI eco keyboard backlight fix

On the MSI Vector 16 HX AI A2XWIG, closing and reopening the lid in **eco / Super Battery** mode can leave the keyboard backlight off. Restoring it manually with the brightness key also changes the brightness level.

This service restores the light automatically on lid open:

1. Save the Super Battery flags at EC `0xEB`.
2. Switch to `comfort` and wait one second.
3. Restore the saved flags **before** switching back to `eco`.

The light stays on, with hardware eco / Super Battery restored, including the firmware's 15 W / 15 W CPU limits.

Tested on **Vector 16 HX AI A2XWIG**, BIOS **E15M3IMS.116**, EC **15M3EMS1.113**, with Ubuntu 26.04.1 LTS, kernel 7.0.0-38-generic and msi-ec 0.13. Automatic recovery was verified with Linux running throughout lid close/open.

## Install

Requires Python 3, systemd, [msi-ec](https://github.com/BeardOverflow/msi-ec), and debugfs mounted at `/sys/kernel/debug`.

From the repository directory:

```sh
sudo modprobe ec_sys write_support=1
cat /sys/module/ec_sys/parameters/write_support  # must show Y
sudo test -e /sys/kernel/debug/ec/ec0/io
python3 ./msi-eco-kbd-restore --check

sudo install -D -m 755 msi-eco-kbd-restore /usr/local/libexec/msi-eco-kbd-restore
sudo install -m 644 systemd/msi-eco-kbd-restore.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now msi-eco-kbd-restore.service
```

If `write_support` shows `N`, add `options ec_sys write_support=1` to `/etc/modprobe.d/ec_sys.conf`, reboot, and repeat the checks before installing.

The script writes an EC register and checks the exact model and BIOS version; keep those checks in place. Recovery runs on every lid open in eco / Super Battery, even if the light was intentionally turned off. Avoid changing power modes during the one-second recovery.

Manual recovery and logs:

```sh
sudo /usr/local/libexec/msi-eco-kbd-restore --once
journalctl -u msi-eco-kbd-restore.service -b
```

## Remove

```sh
sudo systemctl disable --now msi-eco-kbd-restore.service
sudo rm /etc/systemd/system/msi-eco-kbd-restore.service /usr/local/libexec/msi-eco-kbd-restore
sudo systemctl daemon-reload
```

Also remove `/etc/modprobe.d/ec_sys.conf` if you created it solely for this workaround.

Related: [msi-ec#699](https://github.com/BeardOverflow/msi-ec/issues/699). License: [MIT](LICENSE).
