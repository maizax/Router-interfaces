<template>
  <div>
    <vuci-form uci-config="example" class="hideFormButtons" :key="componentRerender">
      <vuci-typed-section type="interface" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" rules="ipaddr" />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="netmask" rules="netmask4" />
        </template>
        <template #edit="{ s }">
          <a-button :uci-section="s" size="small" style="margin-right: 15px" type="primary" @click="editInterface(s['.name'])">Edit</a-button>
          <a-popconfirm @confirm="deleteInterface(s['.name'])" placement="left" title="Are you sure?">
            <a-button type="danger" size="small">Delete</a-button>
          </a-popconfirm>
        </template>
      </vuci-typed-section>
      <p style="font-size: 16px">Add interface</p>
      <a style="font-size: 14px">Interface name </a>
      <a-input label="addInterface" v-model="addInterfaceName" style="width: 250px" />
      <a-button type="primary" style="float:right" @click="addInterface">Create</a-button>
    </vuci-form>
    <interface-modal :sid="sid" @whenApplied="load()" @closeModal="editModal=false" :title="interfaceName" v-if="editModal" :modalRerender="modalRerender" />
  </div>
</template>

<script>
import InterfaceModal from './components/InterfaceModal.vue'

export default {
  components: { InterfaceModal },
  data () {
    return {
      columns: [
        { name: 'name', label: 'Interface name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'edit', label: '' }
      ],
      editModal: false,
      componentRerender: 1000,
      modalRerender: 0,
      sid: '',
      addInterfaceName: '',
      interfaceName: ''
    }
  },
  methods: {
    async load () {
      await this.$uci.load('example')
      this.componentRerender++
    },
    addInterface () {
      if (!this.addInterfaceName) return
      if (this.addInterfaceName.match(/^[a-zA-Z0-9_]+$/) === null) {
        this.$message.error('Only numbers and text allowed!')
        return
      }
      if (this.addInterfaceName.length > 20) {
        this.$message.error('Name cannot exceed 20 symbols!')
        return
      }
      const interfaces = this.$uci.sections('example', 'interface')
      for (let i = 0; i < interfaces.length; i++) {
        if (interfaces[i].name === this.addInterfaceName) {
          this.$message.error('Name already used!')
          return
        }
      }
      this.$spin()
      const newSid = this.$uci.add('example', 'interface')
      this.$uci.set('example', newSid, 'proto', 'Static')
      this.$uci.set('example', newSid, 'name', this.addInterfaceName)
      this.$uci.set('example', newSid, 'dhcpSwitch', '1')
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load().then(() => this.showNewInterface())
          this.$spin(false)
          this.addInterfaceName = ''
        })
      })
    },
    showNewInterface () {
      const lastNewItem = this.$uci.sections('example')
      const lastNewItemId = lastNewItem[lastNewItem.length - 1]['.name']
      this.editInterface(lastNewItemId)
    },
    editInterface (id) {
      this.interfaceName = this.$uci.get('example', id, 'name')
      this.sid = id
      this.editModal = true
      this.modalRerender++
    },
    deleteInterface (id) {
      this.$spin()
      this.$uci.del('example', id)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
          this.$spin(false)
        })
      })
    }
  }
}
</script>

<style>
.hideFormButtons > .ant-row {
  display: none;
}
a {
  color: inherit;
  pointer-events: none;
}
</style>
