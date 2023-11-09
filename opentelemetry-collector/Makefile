include $(TOPDIR)/rules.mk

PKG_NAME:=opentelemetry-collector
PKG_VERSION:=0.88.0
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/open-telemetry/opentelemetry-collector/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=8599121f0ef031fec2c129cd8e53af9a5cb7499b5a03cb0fa1efa6c7c6d3af66

HOST_BUILD_DEPENDS:=golang/host
PKG_BUILD_DEPENDS:=opentelemetry-collector/host golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=go.opentelmetry.io/collector
GO_PKG_BUILD_PKG:=go.opentelemetry.io/collector/cmd/builder

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/host-build.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-host-build.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

PKG_UNPACK:=$(HOST_TAR) -C "$(PKG_BUILD_DIR)" --strip-components=1 -xzf "$(DL_DIR)/$(PKG_SOURCE)"

define Package/opentelemetry-collector
    SECTION:=net
    CATEGORY:=Network
    TITLE:=OpenTelemetry Collector
    URL:=https://github.com/open-telemetry/opentelemetry-collector
    DEPENDS:=$(GO_ARCH_DEPENDS)
endef

define Package/opentelemetry-collector/description
The OpenTelemetry Collector offers a vendor-agnostic implementation on how to receive, process and export telemetry data. In addition, it removes the need to run, operate and maintain multiple agents/collectors in order to support open-source telemetry data formats (e.g. Jaeger, Prometheus, etc.) to multiple open-source or commercial back-ends.
endef

define Host/Compile
    	( cd $(HOST_BUILD_DIR)/cmd/builder; \
		$(GO_HOST_VARS) go install $(GO_PKG_BUILD_PKG); )
endef

define Host/Install
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/bin/
	$(INSTALL_BIN) $(GO_HOST_BUILD_DIR)/bin/builder $(STAGING_DIR_HOSTPKG)/bin/builder
endef

define Build/Prepare
	$(PKG_UNPACK)
	mkdir -p $(GO_PKG_BUILD_BIN_DIR)
	$(CP) ./files/builder.yaml $(PKG_BUILD_DIR)/builder.yaml
	$(SED) 's/PKG_VERSION/$(PKG_VERSION)/g' $(PKG_BUILD_DIR)/builder.yaml
	$(SED) 's/PKG_RELEASE/$(PKG_RELEASE)/g' $(PKG_BUILD_DIR)/builder.yaml
	$(SED) 's,GO_PKG_BUILD_BIN_DIR,$(GO_PKG_BUILD_BIN_DIR),g' $(PKG_BUILD_DIR)/builder.yaml
endef

define Build/Compile
	( $(filter-out "-m32", $(GO_PKG_VARS)) builder \
		--config $(PKG_BUILD_DIR)/builder.yaml \
		--ldflags '-linkmode external -extldflags "-static"'; )
endef

define Package/opentelemetry-collector/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/etc/init.d $(1)/etc/config
	$(INSTALL_BIN) $(GO_PKG_BUILD_BIN_DIR)/otelcol $(1)/usr/bin/otelcol
	$(INSTALL_BIN) ./files/otelcol.init $(1)/etc/init.d/otelcol
	$(INSTALL_CONF) ./files/otelcol.yaml $(1)/etc/config/otelcol
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,opentelemetry-collector))
