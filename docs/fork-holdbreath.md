# holdbreath VoCat Fork

Fork: <https://github.com/holdbreath/VoCat>

Branch: `5ber-ultra-euicc-aid`

This fork carries local patches needed to run mainland China SIM cards and a
wider set of consumer eSIM cards on the NAS. It is not a full rewrite of
upstream; every commit is a small, reviewable change layered on upstream
`master`.

## Patch list

Pushed in order on top of upstream `master` (`875ec3d`):

1. `9c2b771` Add 5ber Ultra eUICC AID discovery

   Adds vendor ISD-R AID discovery for 5ber, XeSIM, eSIM.me, LinksField,
   eSTK.me AUX, and GlocalMe/Tianyu cards. Without these AIDs the adapter
   reports "no eUICC" for those cards even though the card is valid.

2. `6eac37c` Allow mainland China SIM cards (unblock MCC 460/461)

   Empties `internal/device/region.go` `BlockedMCCs`. Upstream refuses service
   for mainland China cards; this fork serves them for SMS and Wi-Fi Calling
   testing.

3. `889e7de` Accept Quectel BCD-padded ICCIDs with inline hex nibbles

   Quectel EC20/EC25 expose the EF_ICCID BCD payload as hex. Some cards carry
   stray A-F nibbles inside the field, which upstream rejects. `parseICCIDIdentifier`
   now drops A-F nibbles before validating the digits so an otherwise valid
   card is not rejected.

## Build and deploy

Build a static Linux binary:

```bash
cd /tmp/vocat-build
GOPROXY=https://goproxy.cn,direct GOSUMDB=off \
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 \
go build -trimpath -ldflags "-s -w" -o vocat-linux-amd64 ./cmd/vocat
```

Deploy to the NAS (current host `192.168.1.4`):

```bash
scp vocat-linux-amd64 holdbreath@192.168.1.4:/tmp/vocat-linux-amd64
ssh holdbreath@192.168.1.4 \
  'sudo install -m 0755 -o root -g root /tmp/vocat-linux-amd64 /opt/vocat/bin/vocat && sudo systemctl restart vocat'
```

The frontend `web/dist` is built separately and is not changed by these
patches. Production uses the binary only.

## Production binary

The NAS binary at `/opt/vocat/bin/vocat` contains the China SIM unblock and the
ICCID BCD fix. It was built before the full vendor AID set from `9c2b771` was
merged into this branch, so 5ber Ultra AID discovery on the NAS depends on a
rebuild from this branch when it is needed.
