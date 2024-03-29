<template>
  <div class="process-wrap" ref="container">
    <scroll>
      <div
        class="block"
        v-for="(item, index) in processList"
        :key="index"
        :class="{ active: item.active }"
        :data-index="index"
        @mousedown="
          (e) => {
            onTap(e, item);
          }
        "
      >
        <span>{{ item.label }}</span>

        <i
          class="el-icon-close"
          v-if="index > 0"
          :class="{ active: index > 0 }"
          @mousedown.stop="onDel(index)"
        ></i>
      </div>
    </scroll>

    <ul v-show="menu.visible" :style="menu.style" class="contextmenu">
      <li @click="onClose('current')" v-if="isHit">关闭当前</li>
      <li @click="onClose('other')">关闭其他</li>
      <li @click="onClose('all')">关闭所有</li>
    </ul>
  </div>
</template>

<script>
import { computed, reactive, ref, onMounted, toRefs } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
export default {
  setup() {
    const store = useStore();
    const router = useRouter();

    const container = ref(null);
    let state = reactive({
      menu: {
        visible: false,
        current: {},
        style: {
          left: 0,
          top: 0,
        },
      },
      isHit: false,
    });

    const processList = computed(() => {
      return store.getters.processList;
    });

    onMounted(() => {
      container.value.oncontextmenu = () => {
        return false;
      };
      document.body.addEventListener("click", () => {
        if (state.menu.visible) {
          state.menu.visible = false;
        }
      });
    });

    const onTap = (e, item) => {
      state.isHit = item.active;
      if (e.button == 0) {
        router.push(item.value);
      } else {
        state.menu = {
          current: item,
          visible: true,
          style: {
            left: e.layerX + "px",
            top: e.layerY + "px",
          },
        };
      }
    };

    const onDel = (index) => {
      store.commit("process/DEL_PROCESS", index);
      toPath();
    };

    const toPath = () => {
      const active = processList.value.find((e) => e.active);
      if (!active) {
        const next = processList.value[processList.value.length - 1];
        router.push(next ? next.value : "/");
      }
    };

    const onClose = (name) => {
      const { current } = state.menu;
      switch (name) {
        case "current":
          onDel(processList.value.findIndex((e) => e.value == current.value));
          break;

        case "other":
          store.commit(
            "process/SET_PROCESS",
            processList.value.filter(
              (e) => e.value == current.value || e.value == "/"
            )
          );
          break;

        case "all":
          store.commit(
            "process/SET_PROCESS",
            processList.value.filter((e) => e.value == "/")
          );
          break;
      }

      toPath();
    };

    return {
      container,
      ...toRefs(state),
      processList,
      onTap,
      onDel,
      onClose,
    };
  },
};
</script>

<style lang="scss" scoped>
.process-wrap {
  display: flex;
  height: 30px;
  position: relative;
  padding: 0 10px;

  .process__left,
  .process__right {
    background-color: #fff;
    height: 30px;
    line-height: 30px;
    padding: 0 2px;
    border-radius: 3px;
    cursor: pointer;

    &:hover {
      background-color: #eee;
    }
  }

  .process__left {
    margin-right: 10px;
  }

  .process__right {
    margin-left: 10px;
  }

  .scroll {
    flex: 1;

    :deep .scroll-wrapper {
      display: flex;
    }
  }

  .block {
    display: inline-block;
    border-radius: 3px;
    height: 30px;
    line-height: 30px;
    padding: 0 10px;
    background-color: #fff;
    font-size: 12px;
    margin-right: 10px;
    color: #909399;
    cursor: pointer;

    &:last-child {
      margin-right: 0;
    }

    i {
      font-size: 14px;
      position: relative;
      top: 3px;
      width: 0;
      overflow: hidden;
      transition: all 0.3s;

      &:hover {
        color: #fff;
        background-color: #fff;
      }
    }

    &:hover {
      .el-icon-close {
        width: auto;
        margin-left: 5px;
      }
    }

    &.active {
      span {
        color: #4165d7;
      }

      i {
        width: auto;
        margin-left: 5px;
      }

      &:before {
        background-color: #4165d7;
      }
    }
  }

  .contextmenu {
    margin: 0;
    background: #fff;
    z-index: 100;
    position: absolute;
    list-style-type: none;
    padding: 5px 0;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 400;
    color: #333;
    box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, 0.3);

    li {
      margin: 0;
      padding: 7px 16px;
      cursor: pointer;

      &:hover {
        background: #eee;
      }
    }
  }
}
</style>