<template>
  <draggable
    v-model="items"
    :group="content.group || 'default-group'"
    :itemKey="getItemKey"
    :disabled="isEditing"
    :animation="200"
    :style="{ ...$attrs.style, ...layoutStyle }"
    @add="onAdd"
    @remove="onRemove"
    @update="onUpdate"
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
    content: {
      type: Object,
      required: true,
    },
  },
  emits: ["trigger-event"],
  setup() {
    return {
      layoutStyle: wwLib.useLayoutStyle()
    };
  },
  computed: {
    items: {
      get() {
        return wwLib.wwCollection.getCollectionData(this.content.data) || [];
      },
      set(value) {
        // Garde le set pour la réactivité locale du v-model
        this.updateList(value);
      }
    },
    getItemKey() {
      // Fonction pour générer une clé unique pour chaque item
      return (item) => {
        return item.id || item._id || JSON.stringify(item);
      };
    },
    isEditing() {
      // Check si on est en mode édition WeWeb
      return wwLib.getFrontWindow().wwLib?.isEditing || false;
    }
  },
  methods: {
    emitListUpdate(eventType = 'change') {
      // Utilise nextTick pour s'assurer que le DOM est mis à jour
      this.$nextTick(() => {
        const currentItems = wwLib.wwCollection.getCollectionData(this.content.data) || [];
        
        console.log('🎯 Emitting update:list event', {
          type: eventType,
          itemCount: currentItems.length,
          group: this.content.group
        });
        
        this.$emit("trigger-event", {
          name: "update:list",
          event: {
            value: currentItems,
            type: eventType,
            group: this.content.group
          }
        });
      });
    },
    onAdd(evt) {
      this.emitListUpdate('add');
    },
    onRemove(evt) {
      this.emitListUpdate('remove');
    },
    onUpdate(evt) {
      this.emitListUpdate('update');
    },
    handleChange(evt) {
      this.emitListUpdate('change');
    },
    updateList(value) {
      // Cette méthode est appelée par le set() du computed
      this.$emit("trigger-event", {
        name: "update:list",
        event: { value }
      });
    }
  }
};
</script>

<style lang="scss" scoped>
/* Ajoute des styles si nécessaire pour améliorer le drag & drop */
</style>