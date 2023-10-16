<template>
  <BaseModal ref="modal" :scrollable="false" @close="$emit('close')">
    <template v-slot:title>
      <div class="d-flex align-items-center">
        <i class="bi bi-clock fs-4 pe-2" /> Time App Settings
      </div>
    </template>
    <template v-slot:body>
      <div class="row mb-3">
        <div class="col-4">
          <span class="align-middle">Format:</span>
        </div>
        <div class="col-8">
          <div class="input-group">
            <input type="text" class="form-control form-control-sm" v-model="format">
            <CDropdown v-if="nodeStore.activeNode" placement="bottom-end" class="me-2">
              <CDropdownToggle size="sm" class="btn-outline-secondary">
                <i class="bi bi-list" />
              </CDropdownToggle>
              <CDropdownMenu>
                <CListGroup>
                  <button
                    v-for="fp in formatPresets"
                    :key="fp.format"
                    class="dropdown-item list-group-item list-group-item-action"
                    :class="{ active: fp.format === format }"
                    type="button"
                    @click="format = fp.format">
                    <div class="d-flex justify-content-between">
                      <span>{{ fp.format }}</span>
                      <small class="text-muted ms-5">{{ fp.example }}</small>
                    </div>
                    <span v-if="fp.blinking" class="badge rounded-pill text-bg-light">blinking</span>
                  </button>
                </CListGroup>
              </CDropdownMenu>
            </CDropdown>
          </div>
        </div>
      </div>
    </template>
    <template v-slot:footer>
      <button type="button" class="btn btn-light" @click="close">Close</button>
      <button type="button" class="btn btn-primary" @click="save">Save</button>
    </template>
  </BaseModal>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { mapStores } from 'pinia';
import {
  CDropdown,
  CDropdownToggle,
  CDropdownMenu,
  CListGroup,
} from '@coreui/vue';
import BaseModal from '@/components/coreui/BaseModal.vue';
import { useNodeStore } from '@/stores/node';
import { useAwtrixStore } from '@/stores/awtrix';

export default defineComponent({
  name: 'AppTimeSettingsModal',
  emits: ['close', 'toast'],
  components: {
    CDropdown,
    CDropdownToggle,
    CDropdownMenu,
    CListGroup,
    BaseModal,
  },
  data() {
    return {
      format: ref<string>(''),
      formatPresets: [
        { format: '%H:%M:%S', example: '13:30:45' },
        { format: '%l:%M:%S', example: '1:30:45 ' },
        { format: '%H:%M', example: '13:30' },
        { format: '%H %M', example: '13.30', blinking: true },
        { format: '%l:%M', example: '1:30' },
        { format: '%l %M', example: '1:30', blinking: true },
        { format: '%l:%M %p', example: '1:30 PM' },
        { format: '%l %M %p', example: '1:30 PM', blinking: true },
      ],
    };
  },
  computed: {
    ...mapStores(
      useNodeStore, // sets this.nodeStore
      useAwtrixStore, // sets this.awtrixStore
    ),
  },
  methods: {
    close() {
      (this.$refs.modal as typeof BaseModal).close();
    },
    save() {
      this.awtrixStore.setSetting('TFORMAT', this.format).then((success: boolean) => {
        if (!success) {
          this.$emit('toast', { title: 'Error', body: 'New time format is not saved :-(' });
        }
      });
    },
  },
  mounted() {
    this.nodeStore.init().then(() => {
      this.format = this.awtrixStore.settings?.TFORMAT || '';
    });
  },
});
</script>

<style scoped>
.dropdown-menu {
  --cui-dropdown-border-width: 0;
  padding-top: 0;
  padding-bottom: 0;
}
.list-group {
  max-height: 260px;
  overflow-y: scroll;
}
.list-group-item.active .text-muted {
  color: var(--cui-list-group-active-color,rgba(255,255,255,.87)) !important;
}
</style>