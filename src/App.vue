<template>
  <!--general web-->
  <div class="web">
    <button>Boton</button>
  </div>
  <div class="w-screen h-screen bg-black bg-opacity-50 fixed top-0 left-0 pointer-events-none">
    <div class="text-white">
      <button
        class="absolute bottom-5 right-5 pointer-events-auto flex flex-row"
        @click="toggleOpen()"
      >
        <div class="chat chat-end" :class="{ flex: !open, hidden: open }">
          <div class="chat-bubble bg-red-400 text-white">
            {{ conversation.bot.welcomeMessage }}
          </div>
        </div>
        <div class="size-20 relative">
          <img
            class="rounded-full"
            :class="{ flex: !open, hidden: open }"
            :src="conversation.bot.img"
            alt="Imagen del bot"
          />
        </div>
      </button>
    </div>

    <!-- chat box -->
    <div class="h-full w-full" :class="{ flex: open, hidden: !open }">
      <div
        class="flex flex-col w-96 h-full absolute top-0 right-0 pointer-events-auto border-2 border-slate-800 resize- overflow-auto"
      >
        <header class="bg-red-500 text-red-50 w-full flex justify-between items-center p-1">
          <div class="flex-grow flex items-center">
            <button @click="toggleOpen()">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                class="icon icon-tabler icons-tabler-outline icon-tabler-minus"
              >
                <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                <path d="M5 12l14 0" />
              </svg>
            </button>
            <div class="bot flex items-center ml-2">
              <img class="size-12 rounded-full" :src="conversation.bot.img" alt="" />
              <span class="ml-2">{{ conversation.bot.userName }}</span>
            </div>
          </div>
          <button class="menu-bar" @click="toggleMenu()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon icon-tabler icons-tabler-outline icon-tabler-baseline-density-small"
            >
              <path stroke="none" d="M0 0h24v24H0z" fill="none" />
              <path d="M4 3h16" />
              <path d="M4 9h16" />
              <path d="M4 15h16" />
              <path d="M4 21h16" />
            </svg>
          </button>
        </header>

        <main class="bg-slate-500 flex-grow overflow-auto p-1">
          <div
            v-for="(message, index) in conversation.messages"
            :key="message.id"
            :ref="index === conversation.messages.length - 1 ? 'lastMessage' : ''"
          >
            <div
              class="chat"
              :class="{ 'chat-start': message.role === 'user', 'chat-end': message.role === 'bot' }"
              v-if="message.type === 'text'"
            >
              <div class="chat-image avatar">
                <div class="w-10 rounded-full">
                  <img v-if="message.role === 'bot'" alt="botImg" :src="getSenderImg(message)" />
                  <img v-else alt="userImg" :src="getSenderImg(message)" />
                </div>
              </div>
              <div class="chat-header m-2">
                {{ getSenderName(message) }}
                <time class="text-xs opacity-50">{{ message.time }}</time>
              </div>
              <div class="chat-bubble" :class="{ 'bg-slate-600': message.role === 'user' }">
                {{ message.text }}
              </div>
            </div>

            <div
              class="chat"
              :class="{ 'chat-start': message.role === 'user', 'chat-end': message.role === 'bot' }"
              v-if="message.type === 'image'"
            >
              <div class="chat-image avatar">
                <div class="w-10 rounded-full">
                  <img v-if="message.role === 'bot'" alt="botImg" :src="getSenderImg(message)" />
                  <img v-else alt="userImg" :src="getSenderImg(message)" />
                </div>
              </div>
              <div class="chat-header m-2">
                {{ getSenderName(message) }}
                <time class="text-xs opacity-50">{{ message.time }}</time>
              </div>
              <div class="chat-bubble" :class="{ 'bg-slate-600': message.role === 'user' }">
                {{ message.text }}

                <img
                  class="rounded-sm m-auto mt-2 mb-2"
                  :src="message.config?.url ?? ''"
                  :alt="message.config?.alt ?? ''"
                />
              </div>
            </div>

            <div
              class="chat"
              :class="{ 'chat-start': message.role === 'user', 'chat-end': message.role === 'bot' }"
              v-if="message.type === 'button'"
            >
              <div class="chat-image avatar">
                <div class="w-10 rounded-full">
                  <img v-if="message.role === 'bot'" alt="botImg" :src="getSenderImg(message)" />
                  <img v-else alt="userImg" :src="getSenderImg(message)" />
                </div>
              </div>
              <div class="chat-header m-2">
                {{ getSenderName(message) }}
                <time class="text-xs opacity-50">{{ message.time }}</time>
              </div>
              <div class="chat-bubble" :class="{ 'bg-slate-600': message.role === 'user' }">
                {{ message.text }}
              </div>
            </div>
          </div>
        </main>

        <div class="flex justify-around items-center bg-slate-500">
          <div
            v-for="button in conversation.messages[conversation.messages.length - 1].config
              ?.buttons"
            :key="button.text"
          >
            <div
              class="btn btn-xs sm:btn-sm md:btn-md btn-active m-2"
              @click="handleButtonClick(button.message)"
            >
              <button class="">
                {{ button.text }}
              </button>
            </div>
          </div>
        </div>

        <footer class="flex h-10">
          <input
            type="text"
            name="chat-text"
            id="chat-text"
            class="bg-white text-black flex-grow"
            v-model="conversationText"
            @keyup.enter="sendMessage()"
            autocomplete="off"
            placeholder="Escribe tu pregunta..."
          />
          <button
            class="h-10 bg-red-400 w-[50px] flex justify-center items-center"
            @click="sendMessage()"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="white"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="icon icon-tabler icons-tabler-outline icon-tabler-arrow-narrow-right bg-red-400"
            >
              <path stroke="none" d="M0 0h24v24H0z" fill="none" />
              <path d="M5 12l14 0" />
              <path d="M15 16l4 -4" />
              <path d="M15 8l4 4" />
            </svg>
          </button>
        </footer>
        <div class="millionbot text-center text-zinc-400 bg-white border-t-2">by 1millionbot</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue'

const open = ref(false)

const conversationText = ref('')

const openChatMenu = ref(false)

interface Bot {
  id: number
  userName: string
  img: string
  welcomeMessage: string
}

interface Message {
  id: number
  text: string
  time: string
  userName: string
  role: string
  type: string
  config?: {
    link?: string
    url?: string
    alt?: string
    buttons?: {
      text: string
      message: string
    }[]
  }
}

interface Recipe {
  id: number
  name: string
  text: string
}

interface Conversation {
  bot: Bot
  user: Bot
  messages: Message[]
}

const toggleOpen = () => {
  open.value = !open.value
}

const toggleMenu = () => {
  console.log('Boton menu apretado')
  openChatMenu.value = !openChatMenu.value
}

const conversation = ref({
  bot: {
    id: 0,
    userName: 'Jonh Bot',
    img: './src/assets/data/botImg.webp',
    welcomeMessage: 'Bienvenido, soy un asistente de cocina'
  },
  user: {
    id: 1,
    userName: 'El lobo feróz',
    img: './src/assets/data/userImg.webp'
  },
  messages: [
    {
      id: 0,
      text: '¡Hola! me llamo Jonh Bot. Te ayudaré con información sobre nuestro servicio de asistentes virtuales.',
      time: '12:45:50',
      userName: '',
      role: 'bot',
      type: 'image',
      config: {
        url: 'http://localhost:5173/botImg.webp',
        alt: 'descripcion de esta imagen'
      }
    },
    {
      id: 1,
      text: 'Hola Jonh Bot, necesito que me aclares unas cuantas cosas',
      time: '12:46:22',
      userName: '',
      role: 'user',
      type: 'text'
    },
    {
      id: 2,
      text: 'Cual es tu pregunta... porque tengo muchas recetas en mi cabeza',
      time: '12:46:28',
      userName: '',
      role: 'bot',
      type: 'button',
      config: {
        buttons: [
          {
            text: 'Tortilla de patatas',
            message: 'Tortilla de patatas'
          },
          {
            text: 'Cordero al horno',
            message: 'Cordero al horno'
          }
        ]
      }
    },
    {
      id: 3,
      text: '¿Me puedes pasar un link del plato?',
      time: '12:46:30',
      userName: '',
      role: 'user',
      type: 'text'
    },
    {
      id: 4,
      text: 'Claro aquí tienes un link con la receta',
      time: '12:46:32',
      userName: '',
      role: 'bot',
      type: 'link',
      config: {
        link: 'http://localhost:5173/botImg.webp'
      }
    }
  ]
} as Conversation)

const getSenderName = (messageObject: Message) => {
  if (messageObject.role === 'bot') {
    return conversation.value.bot.userName
  } else {
    return conversation.value.user.userName
  }
}

const getSenderImg = (messageObject: Message) => {
  if (messageObject.role === 'bot') {
    return conversation.value.bot.img
  } else {
    return conversation.value.user.img
  }
}

const getRandomInt = (max: number) => {
  return Math.floor(Math.random() * Math.floor(max))
}

const scrollToBottom = () => {
  nextTick(() => {
    const lastMessage = document.querySelector('.chat:last-child')
    if (lastMessage) {
      lastMessage.scrollIntoView({ behavior: 'smooth' })
    }
  })
}

const sendMessage = () => {
  const hour = new Date().toLocaleString().split(', ')[1]

  if (conversationText.value === '' || conversationText.value.length <= 0) {
    const messageObject = {
      id: conversation.value.messages.length,
      text: 'Debes escribir algo o no podré ayudarte',
      time: hour,
      userName: conversation.value.bot.userName,
      role: 'bot',
      type: 'text'
    }
    conversation.value.messages.push(messageObject)
  } else {
    const messageObject = {
      id: conversation.value.messages.length,
      text: conversationText.value,
      time: hour,
      userName: conversation.value.user.userName,
      role: 'user',
      type: 'text'
    }
    conversation.value.messages.push(messageObject)

    let x = getRandomInt(4)

    if (x === 0) {
      const messageBot = {
        id: conversation.value.messages.length,
        text: 'Esto es un mensaje de tipo texto',
        time: hour,
        userName: conversation.value.bot.userName,
        role: 'bot',
        type: 'text'
      }
      conversation.value.messages.push(messageBot)
      conversationText.value = ''
    }

    if (x === 1) {
      const messageBot = {
        id: conversation.value.messages.length,
        text: 'Imagen predeterminada',
        time: hour,
        userName: conversation.value.bot.userName,
        role: 'bot',
        type: 'image',
        config: {
          url: 'http://localhost:5173/botImg.webp',
          alt: 'descripcion de esta imagen'
        }
      }
      conversation.value.messages.push(messageBot)
      conversationText.value = ''
    }

    if (x === 2) {
      const messageBot = {
        id: conversation.value.messages.length,
        text: 'Link',
        time: hour,
        userName: conversation.value.bot.userName,
        role: 'bot',
        type: 'link'
      }
      conversation.value.messages.push(messageBot)
      conversationText.value = ''
    }

    if (x === 3) {
      const messageBot = {
        id: conversation.value.messages.length,
        text: '¿Cuál de estas recetas te gustaría hacer?', // QUIERO ENVIAR BOTONES
        time: hour,
        userName: conversation.value.bot.userName,
        role: 'bot',
        type: 'button',
        config: {
          buttons: [
            {
              text: 'Tortilla de patatas',
              message: 'Tortilla de patatas'
            },
            {
              text: 'Cordero al horno',
              message: 'Cordero al horno'
            }
          ]
        }
      }
      conversation.value.messages.push(messageBot)
      conversationText.value = ''
    }
  }
}

const recipe = ref({
  info: [
    {
      id: 0,
      name: 'tortilla de patatas',
      text: `Hola` + '\r\nhola'
    }
  ]
})

// added funtion to handle button clicks
const handleButtonClick = (message: string) => {
  const hour = new Date().toLocaleString().split(', ')[1]
  const newMessage = {
    id: conversation.value.messages.length,
    text: message,
    time: hour,
    userName: '',
    role: 'user',
    type: 'text'
  }
  conversation.value.messages.push(newMessage)
  scrollToBottom()
  if (newMessage.text === 'Tortilla de patatas') {
    const messageBot = {
      id: conversation.value.messages.length,
      text: recipe.value.info[0].text,
      time: hour,
      userName: '',
      role: 'bot',
      type: 'text'
    }
    conversation.value.messages.push(messageBot)
    scrollToBottom()
  }
  if (newMessage.text === 'Cordero al horno') {
    const messageBot = {
      id: conversation.value.messages.length,
      text: 'El cordero al horno se hace asi',
      time: hour,
      userName: '',
      role: 'bot',
      type: 'text'
    }
    conversation.value.messages.push(messageBot)
    scrollToBottom()
  }
  scrollToBottom()
}
</script>

<style scoped></style>
