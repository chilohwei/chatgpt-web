<script lang="ts" setup>
import { HoverButton, SvgIcon } from '@/components/common'
import IconPrompt from '@/icons/Prompt.vue'
import { useAppStore, useChatStore } from '@/store'

defineProps<Props>()

const emit = defineEmits<Emit>()

const { t } = useI18n()

interface Props {
  usingContext?: boolean
  showPrompt: boolean
  searchEnabled?: boolean
  thinkEnabled?: boolean
}

interface Emit {
  (ev: 'export'): void
  (ev: 'toggleUsingContext'): void
  (ev: 'toggleShowPrompt'): void
  (ev: 'toggleSearchEnabled'): void
  (ev: 'toggleThinkEnabled'): void
}

const appStore = useAppStore()
const chatStore = useChatStore()

const collapsed = computed(() => appStore.siderCollapsed)
const currentChatHistory = computed(() => chatStore.getChatRoomByCurrentActive)

function handleUpdateCollapsed() {
  appStore.setSiderCollapsed(!collapsed.value)
}

function onScrollToTop() {
  const scrollRef = document.querySelector('#scrollRef')
  if (scrollRef)
    nextTick(() => scrollRef.scrollTop = 0)
}

function handleExport() {
  emit('export')
}

function toggleUsingContext() {
  emit('toggleUsingContext')
}

function toggleSearchEnabled() {
  emit('toggleSearchEnabled')
}

function toggleThinkEnabled() {
  emit('toggleThinkEnabled')
}

function handleShowPrompt() {
  emit('toggleShowPrompt')
}
</script>

<template>
  <header
    class="sticky top-0 left-0 right-0 z-30 border-b dark:border-neutral-800 bg-white/80 dark:bg-black/20 backdrop-blur-sm"
  >
    <div class="relative flex items-center justify-between min-w-0 overflow-hidden h-14">
      <div class="flex items-center">
        <button
          class="flex items-center justify-center w-11 h-11"
          @click="handleUpdateCollapsed"
        >
          <SvgIcon v-if="collapsed" class="text-2xl" icon="ri:align-justify" />
          <SvgIcon v-else class="text-2xl" icon="ri:align-right" />
        </button>
      </div>
      <h1
        class="flex-1 px-4 pr-6 overflow-hidden cursor-pointer select-none text-ellipsis whitespace-nowrap"
        @dblclick="onScrollToTop"
      >
        {{ currentChatHistory?.title ?? '' }}
      </h1>
      <div class="flex items-center space-x-2">
        <HoverButton @click="handleShowPrompt">
          <span class="text-xl" :class="{ 'text-[#4b9e5f]': usingContext, 'text-[#a8071a]': !usingContext }">
            <IconPrompt class="w-[20px] m-auto" />
          </span>
        </HoverButton>
        <HoverButton :class="{ 'text-[#4b9e5f]': usingContext, 'text-[#a8071a]': !usingContext }" @click="toggleUsingContext">
          <span class="text-xl">
            <SvgIcon icon="ri:chat-history-line" />
          </span>
          <span style="margin-left:.25em">{{ usingContext ? t('chat.showOnContext') : t('chat.showOffContext') }}</span>
        </HoverButton>
        <HoverButton :class="{ 'text-[#4b9e5f]': searchEnabled, 'text-[#a8071a]': !searchEnabled }" @click="toggleSearchEnabled">
          <span class="text-xl">
            <SvgIcon icon="mdi:web" />
          </span>
          <span style="margin-left:.25em">{{ searchEnabled ? t('chat.searchEnabled') : t('chat.searchDisabled') }}</span>
        </HoverButton>
        <HoverButton :class="{ 'text-[#4b9e5f]': thinkEnabled, 'text-[#a8071a]': !thinkEnabled }" @click="toggleThinkEnabled">
          <span class="text-xl">
            <SvgIcon icon="mdi:lightbulb-outline" />
          </span>
          <span style="margin-left:.25em">{{ thinkEnabled ? t('chat.thinkEnabled') : t('chat.thinkDisabled') }}</span>
        </HoverButton>
        <HoverButton @click="handleExport">
          <span class="text-xl text-[#4f555e] dark:text-white">
            <SvgIcon icon="ri:download-2-line" />
          </span>
        </HoverButton>
      </div>
    </div>
  </header>
</template>
