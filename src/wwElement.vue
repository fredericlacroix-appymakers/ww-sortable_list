<template>
  <draggable
    v-model="items"
    :group="content.group || 'default-group'"
    :itemKey="getItemKey"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @change="handleChange"
  >
    <template #item="{ element, index }">
      <div>
        <wwLayoutItemContext is-repeat :data="element" :item="null" :index="index">
          <wwElement v-bind="content.itemContainer"/>
        </wwLayoutItemContext>
      </div>
    </template>
  </draggable>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  components: { draggable },
  props: {
    content: { type: Object, required: true },
  },
  emits: ["trigger-event"],
  setup() {
    return { layoutStyle: wwLib.useLayoutStyle() };
  },
  computed: {
    items: {
      get() {
        return wwLib.wwCollection.getCollectionData(this.content.data) || [];
      },
      set(value) {
        // Garde le set pour la réactivité locale
        this.updateList(value);
      }
    }
  },
  methods: {
    handleChange(evt) {
      // Cette méthode est appelée par vuedraggable lors d'un changement
      this.$nextTick(() => {
        const currentItems = wwLib.wwCollection.getCollectionData(this.content.data) || [];
        this.$emit("trigger-event", {
          name: "update:list",
          event: { 
            value: currentItems,
            change: evt // Info supplémentaire sur le changement
          }
        });
      });
    },
    updateList(value) {
      // Mise à jour de la collection
      this.$emit("trigger-event", {
        name: "update:list",
        event: { value }
      });
    }
  }
};
</script>