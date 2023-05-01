<template>
  <q-page class="column items-center justify-evenly" padding>
    <div class="container" ref="containerRef">
      <q-chat-message
        v-for="{ id, role, content, stamp } in chatStore.history"
        :key="id"
        :name="roleToDisplayName[role]"
        :avatar="roleToAvatarLink[role]"
        :sent="role === 'user'"
        :stamp="dayjs(stamp).fromNow()"
        size="8"
      >
        <ChatItem :content="content" />
      </q-chat-message>

      <!-- Waiting for AI -->
      <q-chat-message
        v-if="thinking"
        :name="roleToDisplayName['assistant']"
        :avatar="roleToAvatarLink['assistant']"
      >
        <div style="display: flex; align-items: center">
          <span style="margin-left: 5px">Thinking </span>
          <q-spinner-dots size="1.5rem" />
        </div>
      </q-chat-message>

      <!-- Waiting for system -->
      <q-chat-message
        v-if="executing"
        :name="roleToDisplayName['system']"
        :avatar="roleToAvatarLink['system']"
      >
        <q-spinner-gears size="2rem" />
      </q-chat-message>

      <!-- Waiting for user's decision -->
      <q-chat-message
        v-if="deciding"
        :name="roleToDisplayName['user']"
        :avatar="roleToAvatarLink['user']"
        sent
        size="4"
      >
        <div>
          <q-spinner-rings size="2rem" />
          <q-btn flat @click="moveOn()">Next Task!</q-btn>
        </div>
      </q-chat-message>
    </div>

    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-fab
        v-model="fabRight"
        vertical-actions-align="right"
        color="primary"
        icon="keyboard_arrow_up"
        direction="up"
      >
        <q-fab-action
          label-position="left"
          color="red"
          @click="resetChatHistory()"
          icon="restart_alt"
          label="Reset chat history"
        />
        <q-fab-action
          label-position="left"
          color="red"
          @click="resetAssistant()"
          icon="smart_toy"
          label="Reset assistant"
        />
        <q-fab-action
          label-position="left"
          color="red"
          @click="resetCredentials()"
          icon="key"
          label="Reset credentials"
        />
      </q-fab>
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import { onMounted, ref, watch } from 'vue';
import { storeToRefs } from 'pinia';
import dayjs from 'dayjs';
import { useRouter } from 'vue-router';
import { useQuasar, scroll } from 'quasar';

import { useAssistantStore } from '../stores/assistant';
import { useChatStore } from '../stores/chat';
import ChatItem from '../components/ChatItem.vue';
import { useCredentialStore } from '../stores/credential';

const $q = useQuasar();
const router = useRouter();
const assistantStore = useAssistantStore();
const chatStore = useChatStore();
const credentialStore = useCredentialStore();
const { lastHistoryItem, deciding, thinking, executing } =
  storeToRefs(chatStore);

const roleToDisplayName = {
  user: 'YOU',
  system: 'SYSTEM ⚙️ ',
  assistant: assistantStore.name,
};

const roleToAvatarLink = {
  user: 'https://cdn.leonardo.ai/users/a177f883-dcbe-45d3-ad7a-1bf625956f6e/generations/2357d032-9844-4869-a1f1-ba1b313589b2/Leonardo_Diffusion_A_human_sitting_in_front_of_a_table_using_a_0.jpg',
  system:
    'https://cdn.leonardo.ai/users/a177f883-dcbe-45d3-ad7a-1bf625956f6e/generations/98b14653-69dd-4a59-9ee6-718cea59133a/Leonardo_Diffusion_An_AI_system_stands_in_the_center_of_a_vast_0.jpg',
  assistant:
    'https://cdn.leonardo.ai/users/a177f883-dcbe-45d3-ad7a-1bf625956f6e/generations/c220a5e3-5fce-4481-ba0d-4ce99a056dd9/Leonardo_Diffusion_A_robotic_Agent_GPT_sitting_on_a_desk_surve_0.jpg',
};

// Return to homepage if assistant is not completed
if (!assistantStore.completed) {
  router.push('/');
} else if (!chatStore.hasHistory) {
  chatStore.addBasicPrompt(assistantStore.prompt);
}

const moveOn = async () => {
  deciding.value = false;

  try {
    await chatStore.exec();
    await chatStore.chat([
      {
        role: 'user',
        content: 'Next Command 💬',
      },
    ]);
  } catch (e) {
    if (e instanceof Error) {
      $q.notify({
        type: 'negative',
        message: e.message,
      });
      return;
    }

    $q.notify({
      type: 'negative',
      message: `Unknown error: ${e}`,
    });
  }
};

onMounted(async () => {
  moveOn();
  scrollToBottom();

  try {
    if (
      lastHistoryItem.value?.role &&
      ['user', 'system'].includes(lastHistoryItem.value?.role)
    ) {
      await chatStore.chat();
    }
  } catch (e) {
    if (e instanceof Error) {
      $q.notify({
        type: 'negative',
        message: e.message,
      });
      return;
    }

    $q.notify({
      type: 'negative',
      message: `Unknown error: ${e}`,
    });
  }
});

const containerRef = ref<HTMLDivElement | null>(null);
const scrollToBottom = () => {
  if (containerRef.value) {
    const target = scroll.getScrollTarget(containerRef.value);
    const offset = containerRef.value.scrollHeight;
    const duration = 1000;
    scroll.animVerticalScrollTo(target, offset, duration);
  }
};

watch(
  chatStore,
  () => {
    scrollToBottom();
  },
  {
    deep: true,
  }
);

const fabRight = ref(false);
const resetChatHistory = () => {
  fabRight.value = false;
  chatStore.$reset();
  location.reload();
};
const resetAssistant = () => {
  fabRight.value = false;
  chatStore.$reset();
  assistantStore.$reset();
  location.reload();
};
const resetCredentials = () => {
  fabRight.value = false;
  chatStore.$reset();
  assistantStore.$reset();
  credentialStore.$reset();
  location.reload();
};
</script>

<style scoped lang="scss">
.container {
  @media screen and (max-width: 1024px) {
    width: 100%;
  }

  width: 80%;
}
</style>
