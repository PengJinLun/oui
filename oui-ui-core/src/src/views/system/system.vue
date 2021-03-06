<template>
  <oui-form uci-config="system" @applied="updateHostname">
    <a-tabs>
      <a-tab-pane key="system" :tab="$t('General Settings')">
        <oui-typed-section type="system" :collapsible="false" v-slot="{ s }" :card="false" @change-hostname="hostnameChanged">
          <oui-form-item-dummy :uci-section="s" :label="$t('Local Time')" name="localtime" :load="localTime">
            <template #append>
              <a-button type="primary" size="small" @click="syncTime" style="margin-left: 10px">{{ $t('Synchronise from Local time') }}</a-button>
            </template>
          </oui-form-item-dummy>
          <oui-form-item-input :uci-section="s" :label="$t('Hostname')" name="hostname" required rules="hostname"/>
          <oui-form-item-select :uci-section="s" :label="$t('Timezone')" name="zonename" :options="zoneinfo" initial="UTC" :save="saveTimezone"/>
        </oui-typed-section>
      </a-tab-pane>
      <a-tab-pane key="logging" :tab="$t('Logging')">
        <oui-typed-section type="system" :collapsible="false" v-slot="{ s }" :card="false">
          <oui-form-item-input :uci-section="s" :label="$t('System log buffer size')" name="log_size" placeholder="16" append="kiB" :rules="{type: 'uinteger', max: 128}"/>
          <oui-form-item-input :uci-section="s" :label="$t('External system log server')" name="log_ip" placeholder="0.0.0.0" rules="ip4addr"/>
          <oui-form-item-input :uci-section="s" :label="$t('External system log server port')" name="log_port" placeholder="514" rules="port"/>
          <oui-form-item-select :uci-section="s" :label="$t('External system log server protocol')" name="log_proto" :options="[ 'udp', 'tcp']"/>
          <oui-form-item-input :uci-section="s" :label="$t('Write system log to file')" name="log_file"/>
          <oui-form-item-select :uci-section="s" :label="$t('Log output level')" name="conloglevel" :options="conlogLevels"/>
          <oui-form-item-select :uci-section="s" :label="$t('Cron Log Level')" name="cronloglevel" :options="cronlogLevels"/>
        </oui-typed-section>
      </a-tab-pane>
      <a-tab-pane key="time" :tab="$t('Time Synchronization')">
        <oui-named-section name="ntp" v-slot="{ s }" :card="false">
          <oui-form-item-switch :uci-section="s" :label="$t('Enable NTP client')" name="enable" :load="ntpCliEnabled" :apply="ntpCliEnable"/>
          <oui-form-item-switch :uci-section="s" :label="$t('Provide NTP server')" name="enable_server" depend="enable"/>
          <oui-form-item-list :uci-section="s" :label="$t('NTP server candidates')" name="server" rules="host" depend="enable"/>
        </oui-named-section>
      </a-tab-pane>
    </a-tabs>
  </oui-form>
</template>

<script>

import zoneinfo from '@/plugins/zoneinfo'

export default {
  data () {
    return {
      localTime: '',
      hostname: '',
      conlogLevels: [
        ['8', this.$t('Debug')],
        ['7', this.$t('Info')],
        ['6', this.$t('Notice')],
        ['5', this.$t('Warning')],
        ['4', this.$t('Error')],
        ['3', this.$t('Critical')],
        ['2', this.$t('Alert')],
        ['1', this.$t('Emergency')]
      ],
      cronlogLevels: [
        ['5', this.$t('Debug')],
        ['8', this.$t('Normal')],
        ['9', this.$t('Warning')]
      ]
    }
  },
  computed: {
    zoneinfo () {
      return zoneinfo.map(item => item[0])
    }
  },
  timers: {
    loadLocalTime: { time: 3000, autostart: true, immediate: true, repeat: true }
  },
  methods: {
    hostnameChanged (self) {
      this.hostname = self.model
    },
    loadLocalTime () {
      this.$ubus.call('oui.system', 'time').then(r => {
        const d = new Date(r.time * 1000)
        const year = d.getFullYear()
        const month = d.getMonth() + 1
        const date = d.getDate()
        const hour = d.getHours()
        const min = d.getMinutes()
        const sec = d.getSeconds()
        this.localTime = `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
      })
    },
    syncTime () {
      const time = Math.floor(new Date().getTime() / 1000)
      this.$ubus.call('oui.system', 'time', { time }).then(() => {
        this.loadLocalTime()
        this.$message.success(this.$t('Successfully synchronized from local time'))
      })
    },
    saveTimezone (sid, value) {
      let timezone = 'UTC'

      for (let i = 0; i < zoneinfo.length; i++) {
        if (zoneinfo[i][0] === value) {
          timezone = zoneinfo[i][1]
          break
        }
      }

      this.$uci.set('system', sid, 'zonename', value)
      this.$uci.set('system', sid, 'timezone', timezone)
    },
    ntpCliEnabled () {
      return this.$system.initEnabled('sysntpd')
    },
    ntpCliEnable (self) {
      return new Promise(resolve => {
        if (self.model === '1') {
          this.$system.initStart('sysntpd').then(() => {
            this.$system.initEnable('sysntpd').then(() => {
              resolve()
            })
          })
        } else {
          this.$system.initStop('sysntpd').then(() => {
            this.$system.initDisable('sysntpd').then(() => {
              resolve()
            })
          })
        }
      })
    },
    updateHostname () {
      this.$store.commit('setHostname', this.hostname)
    }
  }
}
</script>
