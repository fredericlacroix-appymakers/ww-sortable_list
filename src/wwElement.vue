<template>
  <draggable
    :list="items"
    :group="content.group || 'default-group'"
    item-key="id"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @change="onChange"
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
    onChange(evt) {
      // evt contient add | update | remove
      const type =
        evt.added ? "add" :
        evt.moved ? "update" :
        evt.removed ? "remove" :
        "unknown";

      // Toujours créer une nouvelle référence
      const value = [...this.items];

      this.$emit("trigger-event", {
        name: "update:list",
        event: {
          type,
          value,
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