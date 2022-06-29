<template>
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <div class="modal-header">
            Editing: {{ name }}
            <button id="close-btn" @click="$emit('close')">x</button>
          </div>
          <div class="modal-body">
            <vuci-form :uci-config="config" @applied="$emit('close')">
              <vuci-named-section :name="sid" v-slot="{ s }">
                <vuci-form-item-select
                  :uci-section="s"
                  :label="'Protocol'"
                  name="proto"
                  :options="proto"
                  initial="static"
                  @change="onSelectChange($event)"
                  required
                />
                <vuci-form-item-input
                  :uci-section="s"
                  :label="'IP address'"
                  name="address"
                  placeholder="192.168.1.1"
                  depend="proto == 'static'"
                  rules="ipaddr"
                  required
                />
                <vuci-form-item-input
                  :uci-section="s"
                  :label="'Netmask'"
                  name="netmask"
                  placeholder="255.255.255.0"
                  depend="proto == 'static'"
                  rules="netmask4"
                  required
                />
                <vuci-form-item-input
                  :uci-section="s"
                  :label="'Gateway'"
                  name="gateway"
                  depend="proto == 'static'"
                  rules="ipaddr"
                />
                <vuci-form-item-list
                  :uci-section="s"
                  :label="'DNS'"
                  name="dns"
                  depend="proto == 'static'"
                  rules="ipaddr"
                />
                <vuci-form-item-switch
                  :uci-section="s"
                  :label="'DHCP'"
                  name="dhcp"
                  initial
                  true-value="1"
                  false-value="0"
                  depend="proto == 'static'"
                />
              </vuci-named-section>
            </vuci-form>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'EditModal',
  props: ['config', 'sid', 'name'],
  data () {
    return {
      proto: ['dhcp', 'static'],
      changedAfterAction: 0,
      sendingSid: '',
      selectedName: ''
    }
  },
  methods: {
    onSelectChange (event) {
      if (event.model === 'dhcp') {
        this.$uci.set(this.config, this.sid, 'dns', '')
        this.$uci.set(this.config, this.sid, 'gateway', '')
        this.$uci.set(this.config, this.sid, 'address', '')
        this.$uci.set(this.config, this.sid, 'netmask', '')
        this.$uci.set(this.config, this.sid, 'dhcp', '')
        this.$uci.set(this.config, this.sid, 'dns', '')
      } else {
        this.$uci.reset()
        this.$uci.set(this.config, this.sid, 'dhcp', '1')
      }
    },
    async load () {
      await this.$uci.load(this.config)
    }
  }
}
</script>
<style>
/* Edit Modal */
.modal-mask {
  position: fixed;
  z-index: 5;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.1);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
  z-index: 8;
}

.modal-container {
  width: 500px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
  font-family: Helvetica, Arial, sans-serif;
  z-index: 10;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-body #update-btn {
  text-align: center;
}
#close-btn {
  float: right;
  padding: 2px 5px 2px 5px;
}

.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
#close-btn {
  font-size: 18px;
  padding: 0px 10px;
}
</style>
