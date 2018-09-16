Files to create firmware with OpenWrt ImageBuilder  
`make image PROFILE=<your profile> PACKAGES="kmod-usb-net-cdc-ether kmod-usb-net-rndis lighttpd lighttpd-mod-cgi luci-mod-admin-full luci-theme-material luci-app-firewall luci-proto-ppp libiwinfo-lua luci-proto-ipv6 rpcd-mod-rrdns pihole-FTL-full pihole pihole-web usb-modeswitch -dnsmasq" FILES=files/ FILES_REMOVE=files_remove`
