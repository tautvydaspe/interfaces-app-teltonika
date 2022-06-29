<template>
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <div class="modal-header">
            Deleting: {{ name }}
            <button id="close-btn" @click="$emit('close')">x</button>
          </div>
          <div class="modal-body">
            <p>Are you sure you want to delete this section?</p>
            <button
          id="addingBtn"
          @click="$emit('delete', sid)"
        >Yes, delete
        </button>
        <button style="background:#1890ff;"
          id="addingBtn"
          @click="$emit('close')"
        >No!
        </button>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'DeleteConfirmation',
  props: ['sid', 'name'],
  data () {
    return {
      changedAfterAction: 0,
      selectItem: ''
    }
  },
  methods: {
    onSelectChange (event) {
      if (event.model === 'dhcp') {
        this.$uci.set(this.config, this.sid, 'address', '')
        this.$uci.set(this.config, this.sid, 'netmask', '')
        this.$uci.set(this.config, this.sid, 'gateway', '')
        this.$uci.set(this.config, this.sid, 'dns', '')
        this.$uci.set(this.config, this.sid, 'dhcp', '')
      } else {
        this.$uci.reset()
      }
    },
    async load () {
      await this.$uci.load(this.config)
    }
  }
}
</script>
