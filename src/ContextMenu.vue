<script lang="ts">
import { defineComponent, h, PropType } from 'vue'
import { ContextMenuGlobalData, ContextMenuPositionData, MenuConstOptions, MenuItem, MenuOptions } from './ContextMenuDefine'
import ContextSubMenuConstructor from './ContextSubMenu.vue';

export default defineComponent({
  name: 'ContextMenu',
  emits: [ 'update:show', 'close' ],
  props: {
    options: {
      type: Object as PropType<MenuOptions>,
      default: null
    },
    show: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      currentShowPos: {
        x: 0,
        y: 0,
      } as ContextMenuPositionData
    }
  },
  mounted() {
    this.updateCurrentShowPos();
    setTimeout(() => {
      document.addEventListener("click", this.close)
      document.addEventListener("contextmenu", this.close)
    }, 100);
  },
  beforeUnmount() {
    document.removeEventListener("click", this.close);
  },
  watch: {
    show(v : boolean) {
      if(v) this.updateCurrentShowPos();
    } 
  },
  render() {
    if (!this.show) return [];

    return h(ContextSubMenuConstructor, {
      items: this.options.items,
      parentItem: {
        maxWidth: this.options.maxWidth || MenuConstOptions.defaultMaxWidth,
        minWidth: this.options.minWidth || MenuConstOptions.defaultMinWidth,
      } as MenuItem,
      options: this.options,
      zIndex: this.options.zIndex || MenuConstOptions.defaultStartZindex,
      globalData: {
        parentPosition: {
          x: 0,
          y: 0,
        },
        screenSize: {
          w: window.innerWidth,
          h: window.innerHeight,
        },
      } as ContextMenuGlobalData,
      position: this.currentShowPos as ContextMenuPositionData,
      onClose: this.onChildrenClose,
      onPreUpdatePos: this.onChildrenUpdatePos,
    });
  },
  methods: {
    updateCurrentShowPos() {
      this.currentShowPos.x = this.options.x;
      this.currentShowPos.y = this.options.y;
    },
    close() {
      this.$emit("update:show", false);
      this.$emit("close");
    },
    onChildrenClose(byUserClick : boolean) {
      if(byUserClick) {
        this.$emit('close');
        this.$emit('update:show', false);
      }
    },
    onChildrenUpdatePos(newPos: ContextMenuPositionData) {
      this.currentShowPos.x = newPos.x;
      this.currentShowPos.y = newPos.y;
    },
  }
})
</script>
