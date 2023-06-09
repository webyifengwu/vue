<template>
  <transition name="list-fade">
    <div class="playlist" v-show="visible" @click.stop="hide">
      <div class="list-wrapper" @click.stop>
        <!-- 功能位 -->
        <div class="list-header">
          <h1 class="title">
            <i class="icon" :class="modeIcon" @click="changeMode"></i>
            <span class="text">{{ modeText }}</span>
            <span class="clear" @click="showConfirm">
              <i class="icon-clear"></i>
            </span>
          </h1>
        </div>
        <!-- 歌曲列表 -->
        <Scroll class="list-content" ref="scrollRef">
          <transition-group ref="listRef" tag="ul" name="list">
            <!-- <ul ref="listRef"> -->
            <li
              class="item"
              v-for="song in playlist"
              :key="song.id"
              @click.stop="selectItem(song)"
            >
              <i class="current" :class="getCurrentIcon(song)"></i>
              <span class="text">{{ song.name }}</span>
              <span class="favorite" @click.stop="togglefavorite(song)">
                <i :class="getFavoriteIcon(song)"></i>
              </span>
              <span class="delete" @click.stop="removeSong(song)">
                <i class="icon-delete"></i>
              </span>
            </li>
            <!-- </ul> -->
          </transition-group>
        </Scroll>

        <!-- 底部关闭键 -->
        <div class="list-footer" @click.stop="hide">
          <span>关闭</span>
        </div>
      </div>
      <Confirm
        text="是否清空播放列表"
        confirmBtnText="清空"
        ref="confirmRef"
        @confirm="clearAll"
      ></Confirm>
    </div>
  </transition>
</template>

<script setup>
import { computed, nextTick, ref, watch } from "vue";
import { useStore } from "vuex";
import useMode from "../../use-mode";
import useFavorite from "../../use-favorite";
import Scroll from "@/components/base/scroll";
import Confirm from "@/components/base/confirm";
const store = useStore();
const scrollRef = ref(null);
const listRef = ref(null);
const visible = ref(false);
const confirmRef = ref(null);
const currentSong = computed(() => store.getters.currentSong);
const playlist = computed(() => store.state.playlist);
const sequenceList = computed(() => store.state.sequenceList);

watch(currentSong, (newSong) => {
  if (!newSong.id || !visible.value) return;
  scrollToCurrent();
});
// hook
const { modeIcon, changeMode, modeText } = useMode();
const { getFavoriteIcon, togglefavorite } = useFavorite();
function getCurrentIcon(song) {
  return song.id  === currentSong.value.id ? "icon-play":""
  // if (currentSong.value.id === song.id) return "icon-play";
}
// 列表点击播放的效果
function selectItem(song) {
  const sequenceListValue = sequenceList.value.slice();
  const index = sequenceListValue.findIndex((item) => item.id === song.id);
  store.commit("setCurrentIndex", index);
  store.commit("setPlayingState", true);
}
//
function scrollToCurrent() {
  let song = currentSong.value;
  let index = playlist.value.findIndex((item) => item.id === song.id);
  // 找对应li元素
  let targetEl = listRef.value.$el.children[index];
  scrollRef.value.scroll.scrollToElement(targetEl, 300);
}
//
function removeSong(song) {
  store.dispatch("removeSong", song);
}
async function  show() {
  visible.value = true;
  // 等待 dom 更新 之后 再进行更新滚动组件
  await nextTick()
  scrollRef.value.scroll.refresh()
  scrollToCurrent()
}
function hide() {
  visible.value = false;
}
function showConfirm() {
  confirmRef.value.show();
}
function clearAll() {
  store.dispatch("clearSongList");
}
defineExpose({
  show,
});
</script>

<style lang="scss" scoped>
.playlist {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: 200;
  background-color: $color-background-d;
  &.list-fade-enter-active,
  &.list-fade-leave-active {
    transition: opacity 0.3s;
    .list-wrapper {
      transition: all 0.3s;
    }
  }
  &.list-fade-enter-from,
  &.list-fade-leave-to {
    // 进入的起点  离开的终点
    opacity: 0;
    .list-wrapper {
      transform: translate(0, 100%);
    }
  }
  .list-wrapper {
    position: fixed;
    left: 0;
    bottom: 0;
    z-index: 210;
    width: 100%;
    background-color: $color-highlight-background;
    .list-header {
      position: relative;
      padding: 20px 30px 10px 20px;
      .title {
        display: flex;
        align-items: center;
        .icon {
          margin-right: 10px;
          font-size: 24px;
          color: $color-theme-d;
        }
        .text {
          flex: 1;
          font-size: $font-size-medium;
          color: $color-text-l;
        }
        .clear {
          @include extend-click();
          .icon-clear {
            font-size: $font-size-medium;
            color: $color-text-d;
          }
        }
      }
    }
    .list-content {
      max-height: 280px;
      overflow: hidden;
      .item {
        display: flex;
        align-items: center;
        height: 40px;
        padding: 0 30px 0 20px;
        overflow: hidden;
        .current {
          flex: 0 0 20px;
          width: 20px;
          font-size: $font-size-small;
          color: $color-theme-d;
        }
        .text {
          flex: 1;
          @include no-wrap();
          font-size: $font-size-medium;
          color: $color-text-l;
        }
        .favorite {
          @include extend-click();
          margin-right: 15px;
          font-size: $font-size-small;
          color: $color-theme;
          .icon-favorite {
            color: $color-sub-theme;
          }
        }
        .delete {
          @include extend-click();
          font-size: $font-size-small;
          color: $color-theme;
          &.disable {
            color: $color-theme-d;
          }
        }
      }
    }
    .list-add {
      width: 140px;
      margin: 20px auto 30px auto;
      .add {
        display: flex;
        align-items: center;
        padding: 8px 16px;
        border: 1px solid $color-text-l;
        border-radius: 100px;
        color: $color-text-l;
        .icon-add {
          margin-right: 5px;
          font-size: $font-size-small-s;
        }
        .text {
          font-size: $font-size-small;
        }
      }
    }
    .list-footer {
      text-align: center;
      line-height: 60px;
      background: $color-background;
      font-size: $font-size-medium-x;
      color: $color-text-l;
    }
  }
}
</style>

