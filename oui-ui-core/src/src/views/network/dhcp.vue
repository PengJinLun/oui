<template>
  <uci-form config="dhcp" @apply="onApply">
    <uci-section :title="$t('Server Settings')" type="dnsmasq">
      <uci-tab :title="$t('General Settings')" name="general">
        <uci-option type="switch" :label="$t('Domain required')" name="domainneeded" initial="1"></uci-option>
        <uci-option type="switch" :label="$t('Authoritative')" name="authoritative" initial="1"></uci-option>
        <uci-option type="input" :label="$t('Local server')" name="local" required></uci-option>
        <uci-option type="input" :label="$t('Local domain')" name="domain" required></uci-option>
        <uci-option type="switch" :label="$t('Log queries')" name="logqueries" initial="1"></uci-option>
        <uci-option type="dlist" :label="$t('DNS forwardings')" name="server"></uci-option>
        <uci-option type="switch" :label="$t('Rebind protection')" name="rebind_protection" initial="1"></uci-option>
        <uci-option type="switch" :label="$t('Allow localhost')" name="rebind_localhost" depend="rebind_protection"></uci-option>
        <uci-option type="switch" :label="$t('Domain whitelist')" name="rebind_domain" depend="rebind_protection"></uci-option>
      </uci-tab>
      <uci-tab :title="$t('Resolv and Hosts Files')" name="files">
        <uci-option type="switch" :label="$t('Use /etc/ethers')" name="readethers"></uci-option>
        <uci-option type="input" :label="$t('Leasefile')" name="leasefile"></uci-option>
        <uci-option type="switch" :label="$t('Ignore resolve file')" name="noresolv"></uci-option>
        <uci-option type="input" :label="$t('Resolve file')" name="resolvfile" depend="noresolv == '0'"></uci-option>
        <uci-option type="switch" :label="$t('Ignore /etc/hosts')" name="nohosts"></uci-option>
        <uci-option type="dlist" :label="$t('Additional Hosts files')" name="addnhosts"></uci-option>
      </uci-tab>
      <uci-tab :title="$t('Advanced Settings')" name="advanced">
        <uci-option type="switch" :label="$t('Suppress logging')" name="quietdhcp"></uci-option>
        <uci-option type="switch" :label="$t('Allocate IP sequentially')" name="sequential_ip"></uci-option>
        <uci-option type="switch" :label="$t('Filter private')" name="boguspriv"></uci-option>
        <uci-option type="switch" :label="$t('Filter useless')" name="filterwin2k"></uci-option>
        <uci-option type="switch" :label="$t('Localise queries')" name="localise_queries"></uci-option>
        <uci-option type="switch" :label="$t('Expand hosts')" name="expandhosts" initial="1"></uci-option>
        <uci-option type="switch" :label="$t('No negative cache')" name="nonegcache"></uci-option>
        <uci-option type="switch" :label="$t('Strict order')" name="strictorder"></uci-option>
        <uci-option type="dlist" :label="$t('Bogus NX Domain Override')" name="bogusnxdomain"></uci-option>
        <uci-option type="input" :label="$t('DNS server port')" name="port" placeholder="53" rules="port"></uci-option>
        <uci-option type="input" :label="$t('DNS query port')" name="queryport" placeholder="any" rules="port"></uci-option>
        <uci-option type="input" :label="$t('Max. DHCP leases')" name="dhcpleasemax" placeholder="unlimited" rules="uinteger"></uci-option>
        <uci-option type="input" :label="$t('Max. EDNS0 packet size')" name="ednspacket_max" placeholder="1280" rules="uinteger"></uci-option>
        <uci-option type="input" :label="$t('Max. concurrent queries')" name="dnsforwardmax" placeholder="150" rules="uinteger"></uci-option>
      </uci-tab>
    </uci-section>
    <uci-section :title="$t('DHCP Server')" type="dhcp">
      <uci-tab :title="$t('General Settings')" name="general">
        <uci-option type="dummy" :label="$t('Interface')" name="interface"></uci-option>
        <uci-option type="switch" :label="$t('Ignore interface')" name="ignore"></uci-option>
        <uci-option type="input" :label="$t('dhcp-Start')" name="start" placeholder="100" rules="uinteger"></uci-option>
        <uci-option type="input" :label="$t('dhcp-Limit')" name="limit" placeholder="150" rules="uinteger"></uci-option>
        <uci-option type="input" :label="$t('Leasetime')" name="leasetime" placeholder="12h"></uci-option>
      </uci-tab>
      <uci-tab :title="$t('Advanced Settings')" name="advanced">
        <uci-option type="switch" :label="$t('Dynamic DHCP')" name="dynamicdhcp" initial="1"></uci-option>
        <uci-option type="switch" :label="$t('dhcp-Force')" name="force"></uci-option>
        <uci-option type="input" :label="$t('IPv4-Netmask')" name="netmask" rules="ip4addr"></uci-option>
      </uci-tab>
    </uci-section>
    <uci-section :title="$t('Static Leases')" type="host" addable table>
      <uci-option type="input" :label="$t('Hostname')" name="name" rules="hostname"></uci-option>
      <uci-option type="input" :label="$t('MAC-Address')" name="mac" required rules="macaddr"></uci-option>
      <uci-option type="input" :label="$t('IPv4-Address')" name="ip" required rules="ip4addr"></uci-option>
    </uci-section>
  </uci-form>
</template>

<script>
export default {
  methods: {
    onApply() {
      this.$system.initRestart('dnsmasq');
    }
  }
}
</script>