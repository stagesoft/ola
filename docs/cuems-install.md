# OLA 0.10.9 + CUEMS patches — Install instructions

## Extract

```bash
tar xzf ola-0.10.9.nojsmin-2+cuems2.tar.gz
```

## Install

```bash
sudo dpkg -i libola1_0.10.9.nojsmin-2+cuems2_amd64.deb \
             ola_0.10.9.nojsmin-2+cuems2_amd64.deb \
             ola-python_0.10.9.nojsmin-2+cuems2_all.deb \
             libola-dev_0.10.9.nojsmin-2+cuems2_amd64.deb
```

If there are missing dependencies, fix them with:

```bash
sudo apt-get install -f
```

## Pin version

Prevent `apt upgrade` from replacing the patched packages with stock OLA:

```bash
sudo tee /etc/apt/preferences.d/ola-pinned << 'EOF'
Package: ola libola1 libola-dev ola-python
Pin: version 0.10.9.nojsmin-2+cuems2
Pin-Priority: 1001
EOF
```

## systemd service and DMX configuration

OLA's systemd unit, dpkg-divert, and the `cuems-ola-profile` hardware configuration
script are all managed by the **cuems-common** package
(https://github.com/stagesoft/cuems-common).

Install or rebuild cuems-common first:

```bash
sudo apt install cuems-common
```

This provides:

- `olad.service` — runs olad as `olad` user with `--config-dir /etc/ola`
- `dpkg-divert` on `/etc/init.d/olad` — prevents the OLA postinst from spawning a rogue olad
- Symlink `~olad/.ola → /etc/ola` — ensures any auto-started olad uses the system config
- `cuems-ola-profile` — configures OLA plugins per DMX hardware type

Start the service:

```bash
sudo systemctl enable --now olad.service
```

## Configure DMX hardware profile

```bash
sudo cuems-ola-profile pro       # Enttec DMX USB Pro, DMXking, Nodle, etc.
sudo cuems-ola-profile opendmx   # Enttec Open DMX USB (FTDI-based)
sudo cuems-ola-profile artnet    # Network only (ArtNet + sACN)
```

Then restart OLA to apply:

```bash
sudo systemctl restart olad.service
```

## Verify

```bash
olad --version
dpkg -l | grep ola
```

## Uninstall

```bash
sudo apt-get remove --purge ola ola-python libola1 libola-dev
sudo rm /etc/apt/preferences.d/ola-pinned
```
