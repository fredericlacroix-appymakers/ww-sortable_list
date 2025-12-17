<template>
  <draggable
    :list="items"
    :group="content.group || 'default-group'"
    item-key="id"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @add="onAdd"
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
  onAdd(evt) {
    const newIndex = evt?.newIndex ?? null;
    const itemEl = evt?.item;
    const item = itemEl?.__draggable_context?.element 
      ? JSON.parse(JSON.stringify(itemEl.__draggable_context.element))
      : null;

    // Build the new list by inserting item at newIndex
    const currentList = JSON.parse(JSON.stringify(this.items));
    if (item && newIndex !== null) {
      currentList.splice(newIndex, 0, item);
    }

    this.$emit("trigger-event", {
      name: "add:item",
      event: {
        item,
        newIndex,
        list: currentList,
      },
    });
  },

  onRemove(evt) {
    const oldIndex = evt?.oldIndex ?? null;
    // Item already removed from this.items, get from event's item data
    const itemEl = evt?.item;
    const item = itemEl?.__draggable_context?.element 
      ? JSON.parse(JSON.stringify(itemEl.__draggable_context.element))
      : null;

    this.$emit("trigger-event", {
      name: "remove:item",
      event: {
        item,
        oldIndex,
        list: JSON.parse(JSON.stringify(this.items)),
      },
    });
  },

  onUpdate(evt) {
    const oldIndex = evt?.oldIndex ?? null;
    const newIndex = evt?.newIndex ?? null;
    const item = newIndex !== null && this.items[newIndex]
      ? JSON.parse(JSON.stringify(this.items[newIndex]))
      : null;

    this.$emit("trigger-event", {
      name: "update:list",
      event: {
        item,
        oldIndex,
        newIndex,
        list: JSON.parse(JSON.stringify(this.items)),
      },
    });
  },
},
};
</script>







<style lang="scss" scoped>
/* Ajoute des styles si nécessaire pour améliorer le drag & drop */
</style>