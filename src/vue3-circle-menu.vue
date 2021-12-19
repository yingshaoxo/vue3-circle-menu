<script lang="ts">
import { defineComponent, nextTick, onMounted, reactive, ref } from "vue";

export default /*#__PURE__*/ defineComponent({
  name: "Vue3CircleMenu", // vue component name
  components: {},
  props: {
    menuItems: {
      type: Array,
      default: () => [],
    },
    distance: {
      type: Number,
      default: 150,
    },
  },
  setup: (props) => {
    const dict = reactive({
      tempData: {
        baseZIndex: 998,
        triggerReference: ref(null) as any,
        showMenu: false,
        centerX: 0.0,
        centerY: 0.0,
      },
      functions: {
        getTriggerPosition: (triggerReference: any) => {
          if (triggerReference) {
            const { left, top } = triggerReference.getBoundingClientRect();
            dict.tempData.centerX = left + triggerReference.offsetWidth / 2;
            dict.tempData.centerY = top + triggerReference.offsetHeight / 2;
            // console.log(dict.tempData.centerX, dict.tempData.centerY);
          }
        },
        getPointOnCircleByAngle: (angle: number, radius: number) => {
          const x = dict.tempData.centerX + radius * Math.cos(angle);
          const y = dict.tempData.centerY + radius * Math.sin(angle);
          return { x, y };
        },
        getItemPosition: (index: number) => {
          let angle = (index * 0.5 * Math.PI) / props.menuItems.length;
          angle += 1.0 * Math.PI;
          const radius = props.distance;
          return dict.functions.getPointOnCircleByAngle(angle, radius);
        },
        getItemPositionX: (index: number) => {
          const { x } = dict.functions.getItemPosition(index);
          return x;
        },
        getItemPositionY: (index: number) => {
          const { y } = dict.functions.getItemPosition(index);
          return y;
        },
      },
    });

    const closeMenu = () => {
      nextTick(() => {
        setTimeout(() => {
          dict.tempData.showMenu = false;
        }, 100);
      });
    };

    function computeMaxZIndex() {
      function getMaxZIndex(parent: any, current_z = 0) {
        const z =
          parent.style.zIndex != "" ? parseInt(parent.style.zIndex, 10) : 0;
        if (z > current_z) current_z = z;
        const children = parent.shadowRoot
          ? parent.shadowRoot.children
          : parent.children;
        for (let i = 0; i < children.length; i++) {
          const child = children[i];
          current_z = getMaxZIndex(child, current_z);
        }
        return current_z;
      }
      return getMaxZIndex(document.body) + 1;
    }

    onMounted(() => {
      dict.functions.getTriggerPosition(dict.tempData.triggerReference);

      const maxZIndex = computeMaxZIndex();
      dict.tempData.baseZIndex = maxZIndex + 1;

      window.addEventListener("resize", () => {
        dict.functions.getTriggerPosition(dict.tempData.triggerReference);
      });
    });

    return {
      dict,
      props,
      closeMenu,
    };
  },
});
</script>

<template>
  <div
    :style="{
      width: '100vw',
      height: '100vh',
      position: 'fixed',
      left: '0px',
      top: '0px',
      zIndex: dict.tempData.baseZIndex,
      backgroundColor: 'black',
      opacity: 0.0,
    }"
    @click="closeMenu"
    v-if="dict.tempData.showMenu"
  ></div>
  <div
    :ref="
      (el) => {
        dict.tempData.triggerReference = el;
      }
    "
    @click="
      () => {
        dict.functions.getTriggerPosition(dict.tempData.triggerReference);
        dict.tempData.showMenu = !dict.tempData.showMenu;
      }
    "
  >
    <slot name="default"></slot>
  </div>

  <div
    v-for="(item, index) in props.menuItems"
    :key="index"
    :style="{
      left: dict.tempData.showMenu
        ? dict.functions.getItemPositionX(index) + 'px'
        : dict.tempData.centerX + 'px',
      top: dict.tempData.showMenu
        ? dict.functions.getItemPositionY(index) + 'px'
        : dict.tempData.centerY + 'px',
      visibility: dict.tempData.showMenu ? 'visible' : 'hidden',
      position: 'absolute',
      transitionProperty: 'left, top',
      transitionDuration: '1s',
      zIndex: dict.tempData.baseZIndex + 1,
    }"
    @click="closeMenu"
  >
    <slot :name="item"></slot>
  </div>
</template>

<style lang="scss" scoped></style>
