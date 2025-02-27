<template>
  <div class="container mx-auto">
    <div class="grid gap-8 grid-cols-1 lg:grid-cols-2">
      <div>
        <h1 class="text-3xl mb-8 font-bold">
          Collaborative Editing with Collaboration
        </h1>

        <p class="text-gray-700">Just enter your App ID (or URL, if on-premise) and secret to get started. ✨</p><br/>
        <p class="text-gray-700">The JWT is generated client-side and NEVER leaves your device.</p><br/>
        <p class="text-gray-700">Click <a href="https://www.tiptap.dev/docs/cloud" class="underline">here</a> to open
          the documentation.</p>
      </div>

      <div>

        <p>Current mode: {{ mode }}. <span @click="switchMode" class="cursor-pointer underline">Click here to switch to {{
            alternativeMode
          }}</span>
        </p><br/>
        <p>AI: {{ aiEnabled ? 'enabled' : 'disabled' }}. <span @click="aiEnabled = !aiEnabled"
                                                               class="cursor-pointer underline">Click here to toggle.</span>
        </p><br/>

        <div class="grid gap-4">
          <hr/>

          <h2>Collaboration settings</h2>
          <div class="grid gap-1" v-if="mode === 'on-premise'">
            <input
              id="appUrl"
              type="text"
              placeholder="App URL (e.g. ws://localhost:8080)"
              v-model="appUrl"
            />
          </div>

          <div class="grid gap-1" v-if="mode === 'cloud'">
            <input
              id="appId"
              type="text"
              placeholder="App ID"
              v-model="appId"
            />
          </div>

          <div class="grid gap-2">
            <input
              id="secret"
              type="text"
              placeholder="App Secret"
              v-model="secret"
            />
          </div>
        </div
        >
        <div v-if="aiEnabled" class="grid gap-4 mt-4">
          <hr/>

          <h2>Ai settings</h2>

          <div class="grid gap-1" v-if="mode === 'on-premise'">
            <input
              id="appUrl"
              type="text"
              placeholder="AI App URL (e.g. http://localhost:8080)"
              v-model="aiUrl"
            />
          </div>

          <div class="grid gap-1" v-if="mode === 'cloud'">
            <input
              id="aiId"
              type="text"
              placeholder="AI App ID"
              v-model="aiId"
            />
          </div>

          <div class="grid gap-2">
            <input
              id="aiSecret"
              type="text"
              placeholder="AI Secret"
              v-model="aiSecret"
            />
          </div>
        </div>

      </div>
    </div>


    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mt-16">
      <Editor
      :mode="mode"
      :appUrl="appUrl"
      :appId="appId"
      :jwt="jwt"
      :aiEnabled="aiEnabled"
      :aiUrl="aiUrl"
      :aiId="aiId"
      :aiJwt="aiJwt"
      ></Editor>

      <Editor
      :mode="mode"
      :appUrl="appUrl"
      :appId="appId"
      :jwt="jwt"
      :aiEnabled="aiEnabled"
      :aiUrl="aiUrl"
      :aiId="aiId"
      :aiJwt="aiJwt"
      ></Editor>
    </div>
  </div>
</template>

<script setup lang="ts">
import Editor from '../components/Editor.vue'
import {
  onMounted, ref, watch,
} from 'vue'
import * as jose from 'jose'

const mode = ref<'cloud' | 'on-premise'>('cloud')
const alternativeMode = ref<'cloud' | 'on-premise'>('on-premise')
const aiEnabled = ref(true)

const appUrl = ref('')
const appId = ref('')
const secret = ref('')
const jwt = ref('')

const aiUrl = ref('')
const aiId = ref('')
const aiSecret = ref('')
const aiJwt = ref('')

watch(secret, async () => {
  // do NOT generate the JWT like this in production, this is just for demoing purposes. The secret MUST be stored on and never leave the server.
  jwt.value = await new jose.SignJWT({
    allowedDocumentNames: ['test1', 'test2'],
  }).setProtectedHeader({alg: 'HS256'})
    .setIssuedAt()
    .sign(new TextEncoder().encode(secret.value))
})

watch(aiSecret, async () => {
  // do NOT generate the JWT like this in production, this is just for demoing purposes. The secret MUST be stored on and never leave the server.
  aiJwt.value = await new jose.SignJWT({}).setProtectedHeader({alg: 'HS256'})
    .setIssuedAt()
    .sign(new TextEncoder().encode(aiSecret.value))
})

onMounted(() => {
  appUrl.value = window.localStorage.getItem('appUrl') ?? ''
  appId.value = window.localStorage.getItem('appId') ?? ''
  mode.value = (window.localStorage.getItem('mode') as typeof mode.value) ?? 'cloud'
  secret.value = window.localStorage.getItem('secret') ?? ''

  aiEnabled.value = window.localStorage.getItem('aiEnabled') === '1'
  aiUrl.value = window.localStorage.getItem('aiUrl') ?? ''
  aiId.value = window.localStorage.getItem('aiId') ?? ''
  aiSecret.value = window.localStorage.getItem('aiSecret') ?? ''

  const urlParams = new URLSearchParams(window.location.search);
  const modeParam = urlParams.get('mode')

  if (modeParam && ['cloud', 'on-premise'].includes(modeParam)) {
    mode.value = modeParam as typeof mode.value
  }
})

watch(mode, () => {
  if (mode.value === 'cloud') {
    alternativeMode.value = 'on-premise'
  } else {
    alternativeMode.value = 'cloud'
  }
})

const switchMode = () => {
  if (mode.value === 'cloud') {
    mode.value = 'on-premise'
  } else {
    mode.value = 'cloud'
  }
}

watch([appUrl, appId, mode, secret, aiEnabled, aiUrl, aiId, aiSecret], () => {
  window.localStorage.setItem('appUrl', appUrl.value)
  window.localStorage.setItem('appId', appId.value)
  window.localStorage.setItem('mode', mode.value)
  window.localStorage.setItem('secret', secret.value)

  window.localStorage.setItem('aiEnabled', aiEnabled ? '1' : '0')
  window.localStorage.setItem('aiUrl', aiUrl.value)
  window.localStorage.setItem('aiId', aiId.value)
  window.localStorage.setItem('aiSecret', aiSecret.value)
})

</script>

<style lang="postcss">
.ProseMirror {
  padding: 1rem;
}

.ProseMirror-focused {
  border: none;
  outline: none;
}

h2 {
  @apply text-lg font-semibold;
}

label {
  @apply text-sm font-medium;
}

input[type="text"] {
  @apply font-mono;
  @apply border-black border-2;
  @apply rounded-xl;
  @apply px-3 py-2;
}

/* Give a remote user a caret */
.collaboration-cursor__caret {
  border-left: 1px solid #0D0D0D;
  border-right: 1px solid #0D0D0D;
  margin-left: -1px;
  margin-right: -1px;
  pointer-events: none;
  position: relative;
  word-break: normal;
}

/* Render the username above the caret */
.collaboration-cursor__label {
  border-radius: 3px 3px 3px 0;
  color: #0D0D0D;
  font-size: 12px;
  font-style: normal;
  font-weight: 600;
  left: -1px;
  line-height: normal;
  padding: 0.1rem 0.3rem;
  position: absolute;
  top: -1.4em;
  user-select: none;
  white-space: nowrap;
}
</style>
