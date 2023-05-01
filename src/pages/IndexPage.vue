<template>
  <q-page class="column items-center justify-evenly" padding>
    <header class="header">
      <div class="title">Welcome to GPT Agent</div>
      <div class="description">
        Still in <b>BETA</b> might have many bugs 🪲
      </div>
    </header>
    <q-card>
      <q-card-section>
        <span><b>Program your AI</b></span>
        <q-input
          filled
          v-model="name"
          label="Name of your AI"
          hint="Give your AI a name"
          :hide-hint="true"
        />
        <q-input
          filled
          v-model="role"
          label="What is the Purpose?"
          hint="Describe the purpose of your AI"
          :hide-hint="true"
        />
        <span>Set Goals for your AI</span>
        <div class="goal" v-for="(goal, index) in goals" :key="`goal_${index}`">
          <q-input filled v-model="goals[index]" :label="`Goal ${index + 1}`" />
          <q-icon
            size="1.6rem"
            color="primary"
            name="add_circle_outline"
            @click="assistantStore.addGoal()"
          ></q-icon>
          <q-icon
            size="1.6rem"
            color="red"
            name="remove"
            v-show="goals.length > 1"
            @click="assistantStore.removeGoal(index)"
          ></q-icon>
        </div>
      </q-card-section>
      <q-card-actions align="around">
        <q-btn
          flat
          color="secondary"
          title="Fill in demo values"
          @click="assistantStore.setDemoAssistant()"
          >Try it!</q-btn
        >
        <q-btn flat color="dark-page" title="Run AI" @click="run"
          >Run AI!</q-btn
        >
      </q-card-actions>
    </q-card>

    <q-dialog v-model="prompt" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">Please provide credentials</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <q-input
            dense
            v-model="openai"
            label="OpenAI API Key"
            placeholder="sk-..."
            type="text"
          />
          <q-input
            dense
            v-model="google.key"
            label="Google API Key"
            type="text"
          />
          <q-input
            dense
            v-model="google.engine"
            label="Google custom search engine id"
            type="text"
          />
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Add" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
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
          @click="resetAssistant()"
          icon="precision_manufacturing"
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
    <!-- Info -->
    <q-page-sticky position="bottom-left" :offset="[18, 18]">
      <q-fab
        v-model="fabRight"
        vertical-actions-align="left"
        color="primary"
        icon="info_outline"
        direction="up"
      >
        <!-- Github -->
        <q-fab-action
          label-position="left"
          color="red"
          @click="launchWebpage('https://www.youtube.com/watch?v=xvFZjo5PgG0')"
          icon="launch"
          label="Github"
        />
      </q-fab>
    </q-page-sticky>
    <q-page-sticky position="bottom" :offset="[18, 18]">
      <q-fab
        color="transparent-black"
        label="Development Build 1.30"
        hide-icon
        square
      >
      </q-fab>
    </q-page-sticky>
  </q-page>
</template>

<script lang="ts" setup>
import { storeToRefs } from 'pinia';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useAssistantStore } from '../stores/assistant';
import { useCredentialStore } from '../stores/credential';
import { useChatStore } from '../stores/chat';

const router = useRouter();
const assistantStore = useAssistantStore();
const credentialStore = useCredentialStore();
const { name, role, goals, demo } = storeToRefs(assistantStore);
const { openai, google } = storeToRefs(credentialStore);
const chatStore = useChatStore();

const run = () => {
  router.push('/chat');
};

const prompt = ref(credentialStore.requireOpenAICredential);

const fabRight = ref(false);
const resetCredentials = () => {
  fabRight.value = false;
  chatStore.$reset();
  assistantStore.$reset();
  credentialStore.$reset();
  location.reload();
};

const resetAssistant = () => {
  fabRight.value = false;
  chatStore.$reset();
  assistantStore.$reset();
  location.reload();
};
const launchWebpage = (url) => {
  window.open(url, '_blank');
};
</script>

<style scoped lang="scss">
.q-card {
  @media screen and (max-width: 1024px) {
    width: 100%;
  }

  width: 75%;
  background-color: rgb(255, 255, 255);

  .q-input {
    margin: 0.5rem;
    color: red;
  }
}

.header {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: white;

  .title {
    font-size: 2rem;
    font-weight: bold;
  }

  .description {
    font-size: 1rem;
  }
}

.goal {
  display: flex;
  align-items: center;

  .q-input {
    width: 100%;
  }

  .q-icon {
    margin: 0.25rem;
    cursor: pointer;
  }

  .span {
    color: red;
  }
}
</style>
