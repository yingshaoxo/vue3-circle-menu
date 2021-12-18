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
            console.log(dict.tempData.centerX, dict.tempData.centerY);
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

    onMounted(() => {
      dict.functions.getTriggerPosition(dict.tempData.triggerReference);
    });

    const closeMenu = () => {
      nextTick(() => {
        setTimeout(() => {
          dict.tempData.showMenu = false;
        }, 100);
      });
    };

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
    :ref="
        (ref:any) => {
          dict.tempData.triggerReference = ref;
        }
      "
    @click="
      () => {
        dict.functions.getTriggerPosition(dict.tempData.triggerReference);
        dict.tempData.showMenu = !dict.tempData.showMenu;
      }
    "
    @focusout="
      () => {
        closeMenu();
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
      zIndex: 999,
      position: 'absolute',
      transitionProperty: 'left, top',
      transitionDuration: '1s',
    }"
  >
    <slot :name="item"></slot>
  </div>
</template>

<style lang="scss" scoped></style>
