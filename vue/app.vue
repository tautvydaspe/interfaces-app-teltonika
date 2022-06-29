<template>
  <div class="example">
    <vuci-form class="myForm" uci-config="example">
      <vuci-typed-section
        type="interface"
        :columns="columns"
        :key="changedAfterAction"
      >
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="address"
            rules="ipaddr"
          />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="netmask"
            rules="netmask4"
          />
        </template>
        <template #actions="{ s }">
          <a-button :uci-section="s" id="edit" @click="editSection(s)"
            >Edit</a-button
          >
          <a-button :uci-section="s" id="delete" @click="onDeleteClick(s)"
            >Delete</a-button
          >
        </template>
      </vuci-typed-section>
      <div class="adding-wrap">
        <h3>Add Interface</h3>
        <div class="adding">
          <div class="spacing">
            <span>Interface Name:</span>
            <input class="ant-input" type="text" v-model="addingName" />
          </div>
          <button id="addingBtn" @click="handleAdd">Create</button>
        </div>
      </div>
    </vuci-form>
    <EditModal
      v-show="editing"
      :key="changedAfterAction"
      :config="config"
      :sid="sendingSid"
      :name="selectedName"
      @close="closed"
    ></EditModal>
    <DeleteConfirmation
      v-show="deleting"
      :sid="sendingSid"
      :name="selectedName"
      @delete="deleteSection"
      @close="closed"
    ></DeleteConfirmation>
  </div>
</template>

<script>
import EditModal from './components/EditModal.vue'
import DeleteConfirmation from './components/DeleteConfirmation.vue'

export default {
  components: {
    EditModal,
    DeleteConfirmation
  },
  data () {
    return {
      columns: [
        { name: 'name', label: 'Interface Name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'actions', label: 'Actions' }
      ],
      addingName: '',
      config: 'example',
      type: 'interface',
      changedAfterAction: 0,
      editing: false,
      deleting: false,
      selectedName: '',
      sendingSid: '',
      sendingS: {}
    }
  },
  methods: {
    onDeleteClick (sid) {
      this.load()
      this.sendingS = sid
      this.sendingSid = sid['.name']
      this.selectedName = sid.name
      this.deleting = true
    },
    deleteSection (sid) {
      this.$uci.del(this.config, sid)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
        })
      })
      this.deleting = false
    },
    editSection (s) {
      this.load()
      this.sendingS = s
      this.sendingSid = s['.name']
      this.selectedName = s.name
      this.editing = true
    },
    editSectionAfterAdd () {
      const arr = this.$uci.sections(this.config, 'interface')
      this.sendingSid = arr[arr.length - 1]['.name']
      this.selectedName = arr[arr.length - 1].name
      this.load()
      this.editing = true
    },
    handleAdd () {
      if (this.addingName.length > 0 && this.addingName.length < 51) {
        const sid = this.$uci.add(this.config, this.type)
        this.$uci.set(this.config, sid, 'name', this.addingName)
        this.$uci.set(this.config, sid, 'dhcp', '1')
        this.$uci.save().then(() => {
          this.$uci.apply().then(() => {
            this.load().then(() => {
              this.editSectionAfterAdd()
            })
          })
        })
        this.addingName = ''
      } else {
        alert('The length of the new item must be between 1 and 50 characters')
      }
    },
    async load () {
      await this.$uci.load(this.config)
      this.changedAfterAction++
    },
    closed () {
      this.editing = false
      this.deleting = false
      this.changedAfterAction++
    }
  }
}
</script>
<style>
#edit {
  color: #fff;
  background-color: #1890ff;
  border-color: #1890ff;
  text-shadow: 0 -1px 0 rgb(0 0 0 / 12%);
  box-shadow: 0 2px 0 rgb(0 0 0 / 5%);
  margin-right: 5px;
}
#delete {
  color: #fff;
  background-color: #ff4d4f;
  border-color: #ff4d4f;
  text-shadow: 0 -1px 0 rgb(0 0 0 / 12%);
  box-shadow: 0 2px 0 rgb(0 0 0 / 5%);
}
.adding {
  display: flex;
  align-items: center;
}
.spacing {
  display: flex;
}

#addingBtn {
  margin-left: 10px;
  color: #fff;
  background-color: #ff4d4f;
  border-color: #ff4d4f;
  text-shadow: 0 -1px 0 rgb(0 0 0 / 12%);
  box-shadow: 0 2px 0 rgb(0 0 0 / 5%);
  border-radius: 5px;
  padding: 5px 17px 5px 17px;
  border: none;
}
.adding-wrap {
  background: rgb(221, 221, 221);
  padding: 15px;
}

.myForm .ant-form-item-children div {
  display: none;
}
</style>
