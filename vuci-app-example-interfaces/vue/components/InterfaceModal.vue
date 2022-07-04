<template>
<a-modal :title="title" v-model="showModal" class="hideFormFooter" :key="modalRerender" @cancel="$emit('closeModal')">
  <vuci-form uci-config="example" @applied="$emit('whenApplied');checkIfDhcp(sid);">
    <vuci-named-section :name="sid" v-slot="{ s }" :card="false">
      <vuci-form-item-select :uci-section="s" name="proto" initial="Static" label="Protocol" :options="protocolSelection" required />
      <vuci-form-item-input :uci-section="s" name="address" label="Address" placeholder="192.168.1.1" rules="ipaddr" depend="proto == 'Static'" required />
      <vuci-form-item-input :uci-section="s" name="netmask" label="Netmask" placeholder="255.255.255.255" rules="netmask4" depend="proto == 'Static'" required />
      <vuci-form-item-input :uci-section="s" name="gateway" label="Gateway" rules="ipaddr" depend="proto == 'Static'" />
      <vuci-form-item-list :uci-section="s" name="dns" label="DNS" rules="ipaddr" depend="proto == 'Static'" />
      <vuci-form-item-switch :uci-section="s" :initial="true" name="dhcpSwitch" label="DHCP" depend="proto == 'Static'" />
    </vuci-named-section>
  </vuci-form>
  </a-modal>
</template>

<script>
export default {
  props: {
    sid: String,
    modalRerender: Number,
    title: String
  },
  data () {
    return {
      protocolSelection: ['Static', 'DHCP'],
      showModal: true
    }
  },
  methods: {
    checkIfDhcp (id) {
      const checkIfProto = this.$uci.get('example', id, 'proto')
      if (checkIfProto === 'DHCP') {
        this.$uci.set('example', id, 'address')
        this.$uci.set('example', id, 'netmask')
        this.$uci.set('example', id, 'gateway')
        this.$uci.set('example', id, 'dns')
        this.$uci.set('example', id, 'dhcpSwitch')
        this.$uci.save().then(() => {
          this.$uci.apply().then(() => {
            this.$emit('whenApplied')
          })
        })
      }
      if (checkIfProto === 'Static' && (this.$uci.get('example', id, 'dhcpSwitch') === undefined)) {
        this.$uci.set('example', id, 'dhcpSwitch', '1')
        this.$uci.save().then(() => {
          this.$uci.apply().then(() => {
            this.$emit('whenApplied')
          })
        })
      }
    }
  }
}
</script>

<style>
.hideFormFooter .ant-modal-footer {
  border: none;
  display: none;
}
</style>
