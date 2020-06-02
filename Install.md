# Install ThingsPro Edge v1.1.0

## Set to Default

If the unit has been installed prior, we should reset it back to default before installing ThingsPro Edge. The commands listed below is for armhf. For amd64, please re-image the unit.

### Remove docker folder

```sh
rm -rf /overlayfs/docker /overlayfs/working/docker
```

### Reset to default

The command

```sh
mx-set-def
```

> Note: This will wipe out all the data on the device!

## Configure Network
```sh
dhclient eth0
```
> Note: Make sure there is a dhcp server on LAN1

## Download and Install ThingsPro
```sh
wget https://moxaics.s3-ap-northeast-1.amazonaws.com/v3/edge/builds/edge-core-update/release/iotedge/v1.1.0/400/update_1.1.0-400-uc-8112a-me_armhf.deb && \
dpkg -i ./update_1.1.0-400-uc-8112a-me_armhf.deb
```

## Track Installation Progress
```sh
journalctl -u update -f
```

## Reboot Device
```sh
reboot
```
