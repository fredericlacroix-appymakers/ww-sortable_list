<template>
  <draggable
    :list="items"
    :group="content.group || 'default-group'"
    item-key="id"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @add="onChange('add', $event)"
    @update="onChange('update', $event)"
    @remove="onChange('remove', $event)"
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
    // ✅ Source de vérité UNIQUE
    items() {
      return wwLib.wwCollection.getCollectionData(this.content.data) || [];
    },

    // ✅ Désactive le drag en mode édition WeWeb
    isEditing() {
      return wwLib.getFrontWindow().wwLib?.isEditing || false;
    },
  },

  methods: {
    onChange(type, evt) {
      // 🔥 Toujours créer une NOUVELLE référence
      const value = [...this.items];

      this.$emit("trigger-event", {
        name: "update:list",
        event: {
          type,               // 'add' | 'update' | 'remove'
          value,              // état final de la liste
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