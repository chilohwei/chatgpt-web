<script setup lang='ts'>
import { SvgIcon } from '@/components/common'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { useAppStore, useChatStore } from '@/store'
import { useAuthStoreWithout } from '@/store/modules/auth'
import { debounce } from '@/utils/functions/debounce'

const { isMobile } = useBasicLayout()

const appStore = useAppStore()
const chatStore = useChatStore()
const authStore = useAuthStoreWithout()

const loadingRoom = ref(false)

const dataSources = computed(() => chatStore.history)

onMounted(async () => {
  if (authStore.session == null || !authStore.session.auth || authStore.token || authStore.session?.authProxyEnabled)
    await handleSyncChatRoom()
})

async function handleSyncChatRoom() {
  loadingRoom.value = true
  chatStore.syncHistory(() => {
    loadingRoom.value = false
    // 本来这里不需要的, 但是 vue 渲染的时候 chat 可能优先渲染等原因 导致概率不刷新
    if (chatStore.active) {
      const uuid = chatStore.active
      chatStore.syncChat({ uuid } as Chat.History, undefined, () => {
        const scrollRef = document.querySelector('#scrollRef')
        if (scrollRef)
          nextTick(() => scrollRef.scrollTop = scrollRef.scrollHeight)
      })
    }
  })
}

async function handleSelect({ uuid }: Chat.History) {
  if (isActive(uuid))
    return

  // 这里不需要 不然每次切换都rename
  // if (chatStore.active)
  //   chatStore.updateHistory(chatStore.active, { isEdit: false })
  await chatStore.setActive(uuid)

  if (isMobile.value)
    appStore.setSiderCollapsed(true)
}

function handleEdit({ uuid }: Chat.History, isEdit: boolean, event?: MouseEvent) {
  event?.stopPropagation()
  chatStore.updateHistory(uuid, { isEdit })
}

function handleDelete(index: number, event?: MouseEvent | TouchEvent) {
  event?.stopPropagation()
  chatStore.deleteHistory(index)
  if (isMobile.value)
    appStore.setSiderCollapsed(true)
}

const handleDeleteDebounce = debounce(handleDelete, 600)

function handleEnter({ uuid }: Chat.History, isEdit: boolean, event: KeyboardEvent) {
  event?.stopPropagation()
  if (event.key === 'Enter')
    chatStore.updateHistory(uuid, { isEdit })
}

function isActive(uuid: number) {
  return chatStore.active === uuid
}
</script>

<template>
  <NScrollbar class="px-4">
    <NSpin :show="loadingRoom">
      <div class="flex flex-col gap-2 text-sm">
        <template v-if="!dataSources.length">
          <div class="flex flex-col items-center mt-4 text-center text-neutral-300">
            <SvgIcon icon="ri:inbox-line" class="mb-2 text-3xl" />
            <span>{{ $t('common.noData') }}</span>
          </div>
        </template>
        <template v-else>
          <div v-for="(item, index) of dataSources" :key="index">
            <a
              class="relative flex items-center gap-3 px-3 py-3 break-all border rounded-md cursor-pointer hover:bg-neutral-100 group dark:border-neutral-800 dark:hover:bg-[#24272e]"
              :class="isActive(item.uuid) && ['border-[#4b9e5f]', 'bg-neutral-100', 'text-[#4b9e5f]', 'dark:bg-[#24272e]', 'dark:border-[#4b9e5f]', 'pr-14']"
              @click="handleSelect(item)"
            >
              <span>
                <SvgIcon icon="ri:message-3-line" />
              </span>
              <div class="relative flex-1 overflow-hidden break-all text-ellipsis whitespace-nowrap">
                <NInput
                  v-if="item.isEdit"
                  v-model:value="item.title" size="tiny"
                  @keypress="handleEnter(item, false, $event)"
                />
                <span v-else>{{ item.title }}</span>
              </div>
              <div v-if="isActive(item.uuid)" class="absolute z-10 flex visible right-1">
                <template v-if="item.isEdit">
                  <button class="p-1" @click="handleEdit(item, false, $event)">
                    <SvgIcon icon="ri:save-line" />
                  </button>
                </template>
                <template v-else>
                  <button class="p-1">
                    <SvgIcon icon="ri:edit-line" @click="handleEdit(item, true, $event)" />
                  </button>
                  <NPopconfirm placement="bottom" @positive-click="handleDeleteDebounce(index, $event)">
                    <template #trigger>
                      <button class="p-1">
                        <SvgIcon icon="ri:delete-bin-line" />
                      </button>
                    </template>
                    {{ $t('chat.deleteHistoryConfirm') }}
                  </NPopconfirm>
                </template>
              </div>
            </a>
          </div>
        </template>
      </div>
    </NSpin>
  </NScrollbar>
</template>
