<template>
  <el-tabs v-if="radios.length > 0" :value="radios[0].name">
    <el-tab-pane v-for="radio in radios" :key="radio.name" :name="radio.name" :label="radio.name + ` (${radio.hardware})`">
      <uci-form config="wireless">
        <uci-section :name="radio.name">
          <uci-tab :title="$t('General Settings')" name="general">
            <uci-option type="switch" :label="$t('Disabled')" name="disabled"></uci-option>
            <uci-option type="list" :label="$t('Mode')" name="hwmode" :options="radio.hwmodes" required></uci-option>
            <uci-option type="list" :label="$t('Band')" name="htmode" :options="radio.htmodes"></uci-option>
            <uci-option type="list" :label="$t('Channel')" name="channel" :options="radio.channels" :initial="radio.channel" required></uci-option>
            <uci-option type="list" :label="$t('Transmit Power')" name="txpower" :options="radio.txpowerlist" :initial="radio.txpower" required></uci-option>
          </uci-tab>
          <uci-tab :title="$t('Advanced Settings')" name="advanced">
            <uci-option type="list" :label="$t('Country Code')" name="country" :options="radio.countrylist" :initial="radio.country" required></uci-option>
            <uci-option type="input" :label="$t('Distance Optimization')" name="distance" rules="uinteger"></uci-option>
          </uci-tab>
        </uci-section>
        <uci-section :title="$t('Interface')" type="wifi-iface" :option="{radio: radio.name}" :filter="filterInterface" addable>
          <uci-tab :title="$t('General Settings')" name="general">
            <uci-option type="switch" :label="$t('Disabled')" name="disabled"></uci-option>
            <uci-option type="list" :label="$t('Mode')" name="mode" required :options="modes"></uci-option>
            <uci-option type="input" label="SSID" name="ssid" required></uci-option>
            <uci-option type="list" :label="$t('Network')" name="network" :options="interfaces"></uci-option>
            <uci-option type="switch" :label="$t('Hide ESSID')" name="hidden" depend="mode == 'ap'"></uci-option>
            <uci-option type="switch" :label="$t('WMM Mode')" name="wmm" depend="mode == 'ap'" initial="1"></uci-option>
          </uci-tab>
          <uci-tab :title="$t('Wireless Security')" name="security">
            <uci-option type="list" :label="$t('Encryption')" name="encryption" :options="encryptions" initial="none"></uci-option>
            <uci-option type="input" :label="$t('Passphrase')" name="key" depend="encryption != 'none'" password></uci-option>
          </uci-tab>
          <uci-tab :title="$t('MAC-Filter')" name="macfilter">
            <uci-option type="list" :label="$t('Mode')" name="macfilter" :options="macfilters" depend="mode == 'ap'"></uci-option>
            <uci-option type="dlist" :label="$t('MAC-List')" name="maclist" depend="macfilter == 'allow' || macfilter == 'deny'" rules="macaddr"></uci-option>
          </uci-tab>
        </uci-section>
      </uci-form>
    </el-tab-pane>
  </el-tabs>
</template>

<script>
export default {
  data() {
    return {
      radios: [],
      modes: [
        ['ap', this.$t('Access Point')],
        ['sta', this.$t('Client')],
        ['adhoc', this.$t('Ad-Hoc')]
      ],
      interfaces: [],
      encryptions: [
        ['none', this.$t('No encryption')],
        ['psk', 'WPA-PSK'],
        ['psk2', 'WPA2-PSK'],
        ['psk-mixed', 'WPA/WPA2-PSK ' + this.$t('mixed')]
      ],
      macfilters: [
        ['allow', this.$t('Allow listed only')],
        ['deny', this.$t('Allow all except listed')]
      ]
    }
  },
  methods: {
    filterInterface(vm, s) {
      return vm.option.radio === s.device;
    }
  },
  created() {
    const loading = this.$loading({
      text: this.$t('Loading...'),
      spinner: 'el-icon-loading',
      background: 'rgba(0, 0, 0, 0.7)'
    });

    this.$uci.load('wireless').then(() => {
      const sections = this.$uci.sections('wireless', 'wifi-device');
      let radios_num = sections.length;

      sections.forEach(s => {
        const device = s['.name'];
        const batch = [];

        batch.push(['iwinfo', 'info', {device}]);
        batch.push(['iwinfo', 'freqlist', {device}]);
        batch.push(['iwinfo', 'txpowerlist', {device}]);
        batch.push(['iwinfo', 'countrylist', {device}]);

        this.$ubus.callBatch(batch).then(rs => {
          const channels = [['auto', this.$t('Automatic')]];
          const info = rs[0];
          const freqlist = rs[1].results
          const txpowerlist = [];
          const countrylist = [];

          freqlist.forEach(f => {
            if (f.restricted)
              return;
            channels.push([f.channel, `${f.channel} (${f.mhz} MHz)`]);
          });

          rs[2].results.forEach(tx => {
            txpowerlist.push([tx.dbm, `${tx.dbm} dBm (${tx.mw} mW)`]);
          });

          rs[3].results.forEach(c => {
            countrylist.push([c.code, `${c.code} - ${c.country}`]);
          });

          const hwmodes = ['11g'];

          if (info.hwmodes.indexOf('a') > -1 || info.hwmodes.indexOf('ac') > -1)
            hwmodes.push('11a');

          this.radios.push({
            name: device,
            channel: info.channel,
            txpower: info.txpower,
            country: info.country,
            hardware: info.hardware.name,
            hwmodes: hwmodes,
            htmodes: info.htmodes,
            channels: channels,
            txpowerlist: txpowerlist,
            countrylist: countrylist
          });

          radios_num--;

          if (radios_num === 0)
            loading.close();
        });
      });
    });

    this.$network.load().then(() => {
      const interfaces = this.$network.getInterfaces();
      this.interfaces = interfaces.map(item => item.name).filter(name => name !== 'loopback');
    });
  }
}
</script>