<template>
<transition name="mini">
  <div class="mini-player" @click="showFull" v-show="!fullScreen">
    <!-- cd图 -->
    <div class="cd-wrapper">
      <div class="cd" :style="cdStyle">
        <img :src="currentSong.al.picUrl" />
      </div>
    </div>
    <!-- 歌曲信息 -->
    <div class="slider-wrapper">
      <h2 class="name">{{ currentSong.name }}</h2>
      <p class="desc">{{ handle(currentSong) }}</p>
    </div>
    <!-- 进度圆圈 -->
    <div class="control">
      <ProgressCircle :radius="32" :progress="progress">
        <i class="icon-mini" :class="miniBtn" @click.stop="togglePlay"></i>
      </ProgressCircle>
    </div>
    <!-- 播放列表展示按钮-->
    <div class="control" @click.stop="showPlaylist">
      <i class="icon-playlist"></i>
    </div>
    <!-- <div>歌曲列表占位</div> -->
    <Playlist ref="playlistRef"></Playlist>
  </div>
</transition>
</template>

<script setup>
import ProgressCircle from "../progress-circle";
import { useStore } from "vuex";
import { computed, ref } from "vue";
import Playlist from "../playlist";
const props = defineProps(["progress", "cdStyle", "handle", "togglePlay"]);
const store = useStore();
const currentSong = computed(() => store.getters.currentSong);
const playing = computed(() => store.state.playing);
const miniBtn = computed(() =>
  playing.value ? "icon-pause-mini" : "icon-play-mini"
);
const playlistRef = ref(null);
const fullScreen = computed(() => store.state.fullScreen);
function showFull() {
  store.commit("setFullScreen", true);
}

function showPlaylist() {
  playlistRef.value.show();
}
</script>


<style lang="scss" scoped>
.mini-player {
  display: flex;
  align-items: center;
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 180;
  width: 100%;
  height: 60px;
  background: $color-highlight-background;
  .cd-wrapper {
    flex: 0 0 40px;
    width: 40px;
    height: 40px;
    padding: 0 10px 0 20px;
    .cd {
      position: relative;
      height: 100%;
      width: 100%;
      animation: rotate 10s linear infinite;
      overflow: hidden;
      border-radius: 50%;
      img {
        position: absolute;
        width: 130%;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
    }
  }
  .slider-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: center;
    flex: 1;
    line-height: 20px;
    overflow: hidden;
    .name {
      margin-bottom: 2px;
      @include no-wrap();
      font-size: $font-size-medium;
      color: $color-text;
    }
    .desc {
      @include no-wrap();
      font-size: $font-size-small;
      color: $color-text-d;
    }
  }
  .control {
    flex: 0 0 30px;
    width: 30px;
    padding: 0 10px;
    .icon-playlist {
      position: relative;
      top: -2px;
      font-size: 28px;
      color: $color-theme-d;
    }
    .icon-mini {
      position: absolute;
      left: 0;
      top: 0;
      color: $color-theme-d;
      font-size: 32px;
    }
  }
  &.mini-enter-active,
  &.mini-leave-active {
    transition: all 0.6s cubic-bezier(0.45, 0, 0.55, 1);
  }
  &.mini-enter-from,
  &.mini-leave-to {
    opacity: 0;
    transform: translate3d(0, 100%, 0);
  }
}
</style>

