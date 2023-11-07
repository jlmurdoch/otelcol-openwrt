# OpenTelemetry Collector for OpenWRT

## Synopsis
This is a OpenWRT package definition that allows the OpenTelemetry Collector to be compiled for OpenWRT in a small, statically-linked fashion with startup scripts and configuration files. 

It is very minimal build. The builder.yaml can be modified to add more components. It has been tested on both ARM64 and Intel x86 32-bit hosts to compile for a ARM64 target. The build uses musl libc, which is in line with OpenWRT build processes.

It receives data from the hostmetrics, SNMP and OTLP receivers, exporting data over the OTLP HTTP exporter. The configuration is held in otelcol.yaml.

## Prequisites
The following is needed:
- Full OpenWRT source with compiled tools and toolchain
- The following set in the OpenWRT menuconfig:
  - Target System
  - Subtarget
  - Target Profile 
  - Languages  --->
    - Go  --->
      - <*> golang

## Installation
To integrate the source into a working OpenWRT build environment, do as follows:
```
cd /path/to/openwrt
echo "src-git otelcol https://github.com/jlmurdoch/otelcol-openwrt.git" >> feeds.conf.default
./scripts/feeds/update -f otelcol
./scripts/feeds/install -a -p otelcol
```

Next, reconfigure your OpenWRT setup to include the package:
```
make menuconfig
  Network  --->
    <*> opentelemetry-collector
```

Proceed with a full build, otherwise just build this package, as folows:
```
make package/opentelemetry-collector/compile
cp bin/packages/<arch>/base/opentelemetry-collector_<version>_<arch>.ipk ~/
```

## Configuration
At most, the configuration will:
- Collect network metrics
- Add host metadata
- Forward to an OTLP HTTP destination of the users choice

Edit the /etc/config/otelcol.yaml to add the appropriate OTLP HTTP destination.
