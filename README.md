# OpenTelemetry Collector for OpenWRT

## Synopsis
This is an OpenWRT package definition that allows the OpenTelemetry Collector to be cross-compiled on a suitable host for a target OpenWRT base system. The binary is small and statically-linked, with startup scripts and a basic configuration file. 

### Default Build Detail
It is minimal build. The `builder.yaml` can be modified to add more components. It has been tested on both ARM64 and Intel x86 32-bit hosts to compile for an ARM64 target. The build uses musl libc, which is in line with current OpenWRT build processes and keeps static binaries very small in size. The typical binary size is approx 65MB.

Note: this package performs a two-pass compilation, by compiling:
1) OpenTelemetry Collector Builder (`builder`) for the ***host***
  - This is built using the local host-specific toolchain
2) OpenTelemetry Collector binary (`otelcol`) for the ***target***
  - This is built by the builder, using the target-specific toolchain

If one part fails, check to see if it's the builder or the otelcol component.

### Configuration Detail
It receives data from the hostmetrics, SNMP and OTLP receivers, with the resourcedetectionprocessor adding metadata about the host. Exporting data is performed over the ***unconfigured*** OTLP HTTP exporter. The default configuration is held in `otelcol.yaml`.

## Prequisites
The following is needed:
- Full OpenWRT source with compiled/installed tools and toolchain
- The following set in the OpenWRT menuconfig:
  - `Target System`
  - `Subtarget`
  - `Target Profile`
  - `Languages  --->`
    - `Go  --->`
      - `<*> golang`

## Installation
To integrate the source into a working OpenWRT build environment, do as follows:
```
cd /path/to/openwrt
echo "src-git otelcol https://github.com/jlmurdoch/otelcol-openwrt.git" >> feeds.conf.default
./scripts/feeds update otelcol
./scripts/feeds install -a -p otelcol
```

Next, re-run the OpenWRT configuration setup to include the package:
```
make menuconfig
  Network  --->
    <*> opentelemetry-collector
```

It is then possible to proceed with a package-only build as folows:
```
make package/opentelemetry-collector/compile
cp bin/packages/<arch>/base/opentelemetry-collector_<version>_<arch>.ipk ~/
```

## Post-install
The package can be manually installed and initially tested with:
```
opkg install opentelemetry-collector_<version>_<arch>.ipk
service otelcol start
ps | grep otelcol
```

Edit the `/etc/config/otelcol.yaml` to add the appropriate OTLP HTTP destination and restart.
