<template>
  <n-drawer :show="show" height="100%" placement="bottom" :z-index="999999999" :to="`#layout-main`">
    <div class="mv-detail">
      <div
        ref="videoContainerRef"
        class="video-container"
        :class="{ 'cursor-hidden': !showCursor }"
      >
        <video
          ref="videoRef"
          :src="mvUrl"
          class="video-player"
          @ended="handleEnded"
          @timeupdate="handleTimeUpdate"
          @loadedmetadata="handleLoadedMetadata"
          @play="isPlaying = true"
          @pause="isPlaying = false"
          @click="togglePlay"
        ></video>

        <div v-if="autoPlayBlocked" class="play-hint" @click="togglePlay">
          <n-button quaternary circle size="large">
            <template #icon>
              <n-icon size="48">
                <i class="ri-play-circle-line"></i>
              </n-icon>
            </template>
          </n-button>
        </div>

        <div class="custom-controls" :class="{ 'controls-hidden': !showControls }">
          <div class="progress-bar custom-slider">
            <n-slider
              v-model:value="progress"
              :min="0"
              :max="100"
              :tooltip="false"
              :step="0.1"
              @update:value="handleProgressChange"
            ></n-slider>
          </div>

          <div class="controls-main">
            <div class="left-controls">
              <n-tooltip v-if="!props.noList" placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="handlePrev">
                    <template #icon>
                      <n-icon size="24">
                        <n-spin v-if="prevLoading" size="small" />
                        <i v-else class="ri-skip-back-line"></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ t('player.previous') }}
              </n-tooltip>

              <n-tooltip placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="togglePlay">
                    <template #icon>
                      <n-icon size="24">
                        <n-spin v-if="playLoading" size="small" />
                        <i v-else :class="isPlaying ? 'ri-pause-line' : 'ri-play-line'"></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ isPlaying ? t('player.pause') : t('player.play') }}
              </n-tooltip>

              <n-tooltip v-if="!props.noList" placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="handleNext">
                    <template #icon>
                      <n-icon size="24">
                        <n-spin v-if="nextLoading" size="small" />
                        <i v-else class="ri-skip-forward-line"></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ t('player.next') }}
              </n-tooltip>

              <!-- 修复：下载按钮正确嵌套，格式规范 -->
              <n-tooltip placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="handleDownload">
                    <template #icon>
                      <n-icon size="24">
                        <i class="ri-download-line"></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ t('player.download') || '下载MV' }}
              </n-tooltip>

              <div class="time-display">
                {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
              </div>
            </div>

            <div class="right-controls">
              <div v-if="!isMobile" class="volume-control custom-slider">
                <n-tooltip placement="top">
                  <template #trigger>
                    <n-button quaternary circle @click="toggleMute">
                      <template #icon>
                        <n-icon size="24">
                          <i
                            :class="volume === 0 ? 'ri-volume-mute-line' : 'ri-volume-up-line'"
                          ></i>
                        </n-icon>
                      </template>
                    </n-button>
                  </template>
                  {{ volume === 0 ? t('player.unmute') : t('player.mute') }}
                </n-tooltip>
                <n-slider
                  v-model:value="volume"
                  :min="0"
                  :max="100"
                  :tooltip="false"
                  class="volume-slider"
                />
              </div>

              <n-tooltip v-if="!props.noList" placement="top">
                <template #trigger>
                  <n-button quaternary circle class="play-mode-btn" @click="togglePlayMode">
                    <template #icon>
                      <n-icon size="24">
                        <i
                          :class="
                            playMode === 'single' ? 'ri-repeat-one-line' : 'ri-play-list-line'
                          "
                        ></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{
                  playMode === 'single' ? t('player.modeHint.single') : t('player.modeHint.list')
                }}
              </n-tooltip>

              <n-tooltip placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="toggleFullscreen">
                    <template #icon>
                      <n-icon size="24">
                        <i
                          :class="isFullscreen ? 'ri-fullscreen-exit-line' : 'ri-fullscreen-line'"
                        ></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ isFullscreen ? t('player.fullscreen.exit') : t('player.fullscreen.enter') }}
              </n-tooltip>

              <n-tooltip placement="top">
                <template #trigger>
                  <n-button quaternary circle @click="handleClose">
                    <template #icon>
                      <n-icon size="24">
                        <i class="ri-close-line"></i>
                      </n-icon>
                    </template>
                  </n-button>
                </template>
                {{ t('player.close') }}
              </n-tooltip>
            </div>
          </div>
        </div>

        <!-- 添加模式切换提示 -->
        <transition name="fade">
          <div v-if="showModeHint" class="mode-hint">
            <n-icon size="48" class="mode-icon">
              <i :class="playMode === 'single' ? 'ri-repeat-one-line' : 'ri-play-list-line'"></i>
            </n-icon>
            <div class="mode-text">
              {{ playMode === 'single' ? t('player.modeHint.single') : t('player.modeHint.list') }}
            </div>
          </div>
        </transition>
      </div>

      <div class="mv-detail-title" :class="{ 'title-hidden': !showControls }">
        <div class="title">
          <n-ellipsis>{{ currentMv?.name }}</n-ellipsis>
        </div>
      </div>
    </div>
  </n-drawer>
</template>

<script setup lang="ts">
import { NButton, NIcon, NSlider, NTooltip, NSpin, NEllipsis } from 'naive-ui';
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';

// 假设你的API路径正确，若有调整请对应修改
import { getMvUrl } from '@/api/mv';
import { IMvItem } from '@/types/mv';

const { t } = useI18n();
type PlayMode = 'single' | 'auto';
const PLAY_MODE = {
  Single: 'single' as PlayMode,
  Auto: 'auto' as PlayMode
} as const;

const props = withDefaults(
  defineProps<{
    show: boolean;
    currentMv?: IMvItem;
    noList?: boolean;
  }>(),
  {
    show: false,
    currentMv: undefined,
    noList: false
  }
);

const emit = defineEmits<{
  (e: 'update:show', value: boolean): void;
  (e: 'next', loading: (value: boolean) => void): void;
  (e: 'prev', loading: (value: boolean) => void): void;
}>();

// 核心变量定义（修复：补充缺失的cursorTimer定义）
const mvUrl = ref<string>();
const playMode = ref<PlayMode>(PLAY_MODE.Auto);
const videoRef = ref<HTMLVideoElement>();
const videoContainerRef = ref<HTMLElement>();
const isPlaying = ref(false);
const currentTime = ref(0);
const duration = ref(0);
const progress = ref(0);
const bufferedProgress = ref(0);
const volume = ref(100);
const showControls = ref(true);
const isFullscreen = ref(false);
const showModeHint = ref(false);
const autoPlayBlocked = ref(false);
const playLoading = ref(false);
const prevLoading = ref(false);
const nextLoading = ref(false);
const isDragging = ref(false);
const showCursor = ref(true);
const isMobile = computed(() => false); // TODO: 从 settings store 获取 真实的移动端判断逻辑

// 修复：补充缺失的定时器变量定义
let controlsTimer: NodeJS.Timeout | null = null;
let cursorTimer: NodeJS.Timeout | null = null;

/**
 * 格式化时间（秒 -> 00:00 格式）
 */
const formatTime = (seconds: number) => {
  if (!Number.isFinite(seconds)) return '00:00';
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = Math.floor(seconds % 60);
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
};

/**
 * 切换播放/暂停状态
 */
const togglePlay = () => {
  if (!videoRef.value) return;
  if (isPlaying.value) {
    videoRef.value.pause();
  } else {
    videoRef.value.play();
  }
  resetCursorTimer();
};

/**
 * 切换静音/取消静音
 */
const toggleMute = () => {
  if (!videoRef.value) return;
  volume.value = volume.value === 0 ? 100 : 0;
};

/**
 * 进度条变化处理
 */
const handleProgressChange = (value: number) => {
  if (!videoRef.value || !duration.value) return;
  const newTime = (value / 100) * duration.value;
  videoRef.value.currentTime = newTime;
};

/**
 * 视频播放时间更新
 */
const handleTimeUpdate = () => {
  if (!videoRef.value) return;
  currentTime.value = videoRef.value.currentTime;
  if (!isDragging.value) {
    progress.value = (currentTime.value / duration.value) * 100;
  }

  if (videoRef.value.buffered.length > 0) {
    bufferedProgress.value = (videoRef.value.buffered.end(0) / duration.value) * 100;
  }
};

/**
 * 视频元数据加载完成
 */
const handleLoadedMetadata = () => {
  if (!videoRef.value) return;
  duration.value = videoRef.value.duration;
};

/**
 * 重置控制栏显示定时器
 */
const resetControlsTimer = () => {
  if (controlsTimer) {
    clearTimeout(controlsTimer);
  }
  showControls.value = true;
  controlsTimer = setTimeout(() => {
    if (isPlaying.value) {
      showControls.value = false;
    }
  }, 3000);
};

/**
 * 重置鼠标光标显示定时器
 */
const resetCursorTimer = () => {
  if (cursorTimer) {
    clearTimeout(cursorTimer);
  }
  showCursor.value = true;
  if (isPlaying.value && !showControls.value) {
    cursorTimer = setTimeout(() => {
      showCursor.value = false;
    }, 3000);
  }
};

/**
 * 鼠标移动事件处理
 */
const handleMouseMove = () => {
  resetControlsTimer();
  resetCursorTimer();
};

/**
 * 加载MV播放地址
 */
const loadMvUrl = async (mv: IMvItem) => {
  playLoading.value = true;
  autoPlayBlocked.value = false;
  try {
    const res = await getMvUrl(mv.id);
    mvUrl.value = res.data.data.url;
    await nextTick();
    if (videoRef.value) {
      try {
        await videoRef.value.play();
      } catch (error) {
        console.warn('自动播放失败，可能需要用户交互:', error);
        autoPlayBlocked.value = true;
      }
    }
  } catch (error) {
    console.error('加载MV地址失败:', error);
  } finally {
    playLoading.value = false;
  }
};

/**
 * 关闭MV播放器
 */
const handleClose = () => {
  emit('update:show', false);
};

/**
 * 视频播放结束处理
 */
const handleEnded = () => {
  if (playMode.value === PLAY_MODE.Single) {
    // 单曲循环模式，重新加载当前MV
    if (props.currentMv) {
      loadMvUrl(props.currentMv);
    }
  } else {
    // 自动播放模式，触发下一个
    emit('next', (value: boolean) => {
      nextLoading.value = value;
    });
  }
};

/**
 * 切换播放模式（单曲循环/列表播放）
 */
const togglePlayMode = () => {
  playMode.value = playMode.value === PLAY_MODE.Auto ? PLAY_MODE.Single : PLAY_MODE.Auto;
  showModeHint.value = true;
  setTimeout(() => {
    showModeHint.value = false;
  }, 1500);
};

/**
 * 检查全屏API兼容性
 */
const checkFullscreenAPI = () => {
  const doc = document as any;
  return {
    requestFullscreen:
      videoContainerRef.value?.requestFullscreen ||
      (videoContainerRef.value as any)?.webkitRequestFullscreen ||
      (videoContainerRef.value as any)?.mozRequestFullScreen ||
      (videoContainerRef.value as any)?.msRequestFullscreen,
    exitFullscreen:
      doc.exitFullscreen ||
      doc.webkitExitFullscreen ||
      doc.mozCancelFullScreen ||
      doc.msExitFullscreen,
    fullscreenElement:
      doc.fullscreenElement ||
      doc.webkitFullscreenElement ||
      doc.mozFullScreenElement ||
      doc.msFullscreenElement,
    fullscreenEnabled:
      doc.fullscreenEnabled ||
      doc.webkitFullscreenEnabled ||
      doc.mozFullScreenEnabled ||
      doc.msFullscreenEnabled
  };
};

/**
 * 切换全屏状态
 */
const toggleFullscreen = async () => {
  const api = checkFullscreenAPI();

  if (!api.fullscreenEnabled) {
    console.warn('全屏API不可用');
    return;
  }

  try {
    if (!api.fullscreenElement) {
      await videoContainerRef.value?.requestFullscreen();
      isFullscreen.value = true;
    } else {
      await document.exitFullscreen();
      isFullscreen.value = false;
    }
  } catch (error) {
    console.error('切换全屏失败:', error);
  }
};

/**
 * 监听全屏状态变化
 */
const handleFullscreenChange = () => {
  const api = checkFullscreenAPI();
  isFullscreen.value = !!api.fullscreenElement;
};

/**
 * 键盘快捷键处理
 */
const handleKeyPress = (e: KeyboardEvent) => {
  if (e.key === 'f' || e.key === 'F') {
    toggleFullscreen();
  }
};

/**
 * 上一个MV
 */
const handlePrev = () => {
  prevLoading.value = true;
  emit('prev', (value: boolean) => {
    prevLoading.value = value;
  });
};

/**
 * 下一个MV
 */
const handleNext = () => {
  nextLoading.value = true;
  emit('next', (value: boolean) => {
    nextLoading.value = value;
  });
};

/**
 * 完善：MV下载功能（核心新增逻辑）
 */
const handleDownload = async () => {
  if (!mvUrl.value || !props.currentMv) {
    console.warn('下载失败：MV地址或信息不存在');
    return;
  }

  try {
    // 1. 创建下载链接
    const downloadLink = document.createElement('a');
    downloadLink.href = mvUrl.value;
    // 2. 设置下载文件名（使用MV名称，避免中文乱码）
    const fileName = encodeURIComponent(props.currentMv.name || `MV_${Date.now()}`) + '.mp4';
    downloadLink.download = fileName;
    // 3. 触发下载
    document.body.appendChild(downloadLink);
    downloadLink.click();
    // 4. 清理DOM
    document.body.removeChild(downloadLink);

    console.log('下载已触发：', fileName);
  } catch (error) {
    console.error('下载失败：', error);
    alert('MV下载失败，请稍后重试');
  }
};

// 监听音量变化，同步到视频元素
watch(volume, (newVolume) => {
  if (videoRef.value) {
    videoRef.value.volume = newVolume / 100;
  }
});

// 监听currentMv变化，加载对应的MV
watch(
  () => props.currentMv,
  async (newMv) => {
    if (newMv) {
      await loadMvUrl(newMv);
    }
  }
);

// 监听播放状态变化，重置光标状态
watch(isPlaying, (newValue) => {
  if (!newValue) {
    showCursor.value = true;
    if (cursorTimer) {
      clearTimeout(cursorTimer);
    }
  } else {
    resetCursorTimer();
  }
});

// 监听控制栏状态变化，重置光标状态
watch(showControls, (newValue) => {
  if (newValue) {
    showCursor.value = true;
    if (cursorTimer) {
      clearTimeout(cursorTimer);
    }
  } else {
    resetCursorTimer();
  }
});

// 组件挂载：添加所有事件监听（修复：去重，避免重复挂载）
onMounted(() => {
  document.addEventListener('mousemove', handleMouseMove);
  document.addEventListener('fullscreenchange', handleFullscreenChange);
  document.addEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.addEventListener('mozfullscreenchange', handleFullscreenChange);
  document.addEventListener('MSFullscreenChange', handleFullscreenChange);
  document.addEventListener('keydown', handleKeyPress);
});

// 组件卸载：移除所有事件监听（修复：去重，避免内存泄漏）
onUnmounted(() => {
  document.removeEventListener('mousemove', handleMouseMove);
  document.removeEventListener('fullscreenchange', handleFullscreenChange);
  document.removeEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.removeEventListener('mozfullscreenchange', handleFullscreenChange);
  document.removeEventListener('MSFullscreenChange', handleFullscreenChange);
  document.removeEventListener('keydown', handleKeyPress);

  // 清理所有定时器
  if (controlsTimer) clearTimeout(controlsTimer);
  if (cursorTimer) clearTimeout(cursorTimer);
});
</script>

<style scoped lang="scss">
.mv-detail {
  @apply h-full bg-light dark:bg-black;

  &-title {
    @apply fixed top-0 left-0 right-0 p-4 z-10 transition-opacity duration-300;
    background: linear-gradient(to bottom, rgba(0, 0, 0, 0.7), transparent);

    .title {
      @apply text-white text-lg font-bold;
    }
  }
}

.video-container {
  @apply h-full w-full relative;

  .video-player {
    @apply h-full w-full object-contain bg-black;
  }

  .play-hint {
    @apply absolute inset-0 flex items-center justify-center bg-black bg-opacity-50;
    .n-button {
      @apply text-white hover:text-green-500 transition-colors;
    }
  }

  .custom-controls {
    @apply absolute bottom-0 left-0 right-0 p-4 transition-opacity duration-300;
    background: linear-gradient(to top, rgba(0, 0, 0, 0.7), transparent);

    .controls-main {
      @apply flex justify-between items-center;

      .left-controls,
      .right-controls {
        @apply flex items-center gap-2;

        .n-button {
          @apply text-white hover:text-green-500 transition-colors;
        }

        .time-display {
          @apply text-white text-sm ml-4;
        }
      }
    }
  }
}

.mode-hint {
  @apply absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 flex flex-col items-center;

  .mode-icon {
    @apply text-white mb-2;
  }

  .mode-text {
    @apply text-white text-sm;
  }
}

.custom-slider {
  :deep(.n-slider) {
    --n-rail-height: 4px;
    --n-rail-color: theme('colors.gray.200');
    --n-rail-color-dark: theme('colors.gray.700');
    --n-fill-color: theme('colors.green.500');
    --n-handle-size: 12px;
    --n-handle-color: theme('colors.green.500');

    &.n-slider--vertical {
      height: 100%;

      .n-slider-rail {
        width: 4px;
      }

      &:hover {
        .n-slider-rail {
          width: 6px;
        }

        .n-slider-handle {
          width: 14px;
          height: 14px;
        }
      }
    }

    .n-slider-rail {
      @apply overflow-hidden transition-all duration-200;
      @apply bg-gray-500 dark:bg-dark-300 bg-opacity-10 !important;
    }

    .n-slider-handle {
      @apply transition-all duration-200;
      opacity: 0;
    }

    &:hover .n-slider-handle {
      opacity: 1;
    }
  }
}

.progress-bar {
  @apply mb-4;

  .progress-rail {
    @apply relative w-full h-1 bg-gray-600;

    .progress-buffer {
      @apply absolute top-0 left-0 h-full bg-gray-400;
    }
  }
}

.volume-control {
  @apply flex items-center gap-2;

  .volume-slider {
    width: 100px;
  }
}

.controls-hidden {
  opacity: 0;
  pointer-events: none;
}

.cursor-hidden {
  cursor: none;
}

.title-hidden {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
