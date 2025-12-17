<template>
  <draggable
    :list="items"
    :group="content.group || 'default-group'"
    item-key="id"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @remove="onRemove"
    @update="onUpdate"
  >
    <template #item="{ element, index }">
      <div>
        <wwLayoutItemContext
          is-repeat
          :data="element"
          :item="null"
          :index="index"
        >
          <wwElement v-bind="content.itemContainer" />
        </wwLayoutItemContext>
      </div>
    </template>
  </draggable>
</template>




<script>
import draggable from "vuedraggable";

export default {
  components: { draggable },

  props: {
    content: {
      type: Object,
      required: true,
    },
  },

  emits: ["trigger-event"],

  setup() {
    return {
      layoutStyle: wwLib.useLayoutStyle(),
    };
  },

  computed: {
    items() {
      return wwLib.wwCollection.getCollectionData(this.content.data) || [];
    },

    isEditing() {
      return wwLib.getFrontWindow().wwLib?.isEditing || false;
    },
  },

  methods: {
    onRemove(evt) {
      // 🔥 élément retiré (clé pour ton workflow)
      const removedItem = evt?.item?._underlying_vm_ || null;

      this.$emit("trigger-event", {
        name: "remove:item",
        event: {
          item: removedItem,
          oldIndex: evt?.oldIndex ?? null,
        },
      });
    },

    onUpdate(evt) {
      // réorganisation interne (optionnel)
      this.$emit("trigger-event", {
        name: "update:list",
        event: {
          oldIndex: evt?.oldIndex ?? null,
          newIndex: evt?.newIndex ?? null,
        },
      });
    },
  },
};
</script>






<style lang="scss" scoped>
/* Ajoute des styles si nécessaire pour améliorer le drag & drop */
</style>