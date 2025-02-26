<script setup>
import { ref, nextTick, onMounted, watch } from 'vue'

// Config
const config = useRuntimeConfig()
const apiUrl = `${config.public.apiBaseUrl}/api/${config.public.apiVersion}/chat`

const message = ref("")
const clientKey = ref("550e8400-e29b-41d4-a716-446655440000") // Clé client
const messages = ref([]) // Historique des messages
const isLoading = ref(false) // Animation de chargement
const chatContainer = ref(null) // Référence à la zone de chat
const isOpen = ref(false) // Gère l'ouverture du chatbot
const isMobile = ref(false) // Détecte si l'utilisateur est sur mobile
const isChatActive = ref(false) // Gère l'affichage entre Home & Chat
const showOptions = ref(false) // Affiche les options


// Fonction pour jouer un son lorsqu'un message du chatbot arrive
const playNotificationSound = () => {
     const audio = new Audio("/sounds/notification.mp3") // Charger le son
     audio.volume = 0.5 // Ajuste le volume si nécessaire (0.0 - 1.0)
     audio.play().catch(error => console.warn("Impossible de jouer le son :", error)) // Gestion d'erreur
}


// Exemples de questions suggérées
const suggestedQuestions = [
     "Quels sont les meilleurs restaurants ?",
     "Quels sont les événements à venir ?",
     "Quels sites touristiques visiter ?",
     "Où trouver un hôtel ?",
     "Quels sont les meilleurs cafés ?",
     "Quels sont les moyens de transport ?",
     "Quels sont les horaires de la mairie ?"
]

// Fonction pour envoyer une question pré-remplie
const sendSuggestedMessage = (question) => {
  message.value = question
  sendMessage()
}

// Vérifie si c'est un nouveau visiteur
const isBrowser = typeof window !== "undefined"

// Détecte si l'utilisateur est sur mobile
onMounted(() => {
  isMobile.value = window.innerWidth <= 768

  if (isBrowser) {
    const savedMessages = localStorage.getItem("chat_history")
    if (savedMessages) {
      messages.value = JSON.parse(savedMessages)
    }
  }
})

// Sauvegarde l'historique en localStorage
watch(messages, (newMessages) => {
  if (isBrowser) {
    localStorage.setItem("chat_history", JSON.stringify(newMessages))
  }
}, { deep: true })

// Scroll automatique
const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight
    }
  })
}

// Fonction pour vider l'historique
const clearChat = () => {
  messages.value = []
  if (isBrowser) {
    localStorage.removeItem("chat_history")
  }
}


// Envoi d'un message
const sendMessage = async () => {
     if (!message.value.trim()) return

     messages.value.push({ text: message.value, sender: "user" })
     isChatActive.value = true
     scrollToBottom()

     isLoading.value = true
     const userMessage = message.value
     message.value = ""

     try {
          const res = await fetch(apiUrl, {
               method: "POST",
               headers: { "Content-Type": "application/json" },
               body: JSON.stringify({ message: userMessage, clientKey: clientKey.value })
          })
          const data = await res.json()

          setTimeout(() => {
               messages.value.push({ text: data.response, sender: "bot" })
               playNotificationSound() // 🔊 Joue le son quand le chatbot répond
               scrollToBottom()
               isLoading.value = false
          }, 1000)
     } catch (error) {
          messages.value.push({ text: "Erreur : Impossible de contacter le chatbot.", sender: "bot" })
          scrollToBottom()
          isLoading.value = false
     }
}

// Fonction pour ouvrir/fermer le chatbot
const toggleChat = () => {
     isOpen.value = !isOpen.value
     isChatActive.value = false

     // Si on ouvre le chat, scroller automatiquement vers le bas
     if (isOpen.value) {
          nextTick(() => {
               scrollToBottom()
          })
     }
}
// Fonction pour activer/désactiver les notifications
const toggleOptions = () => {
     showOptions.value = !showOptions.value
}

const formatMessage = (text) => {
  // Gérer les sauts de ligne et les listes
  return text
      .replace(/\n/g, "<br>") // Remplace les sauts de ligne par <br>
      .replace(/(\d{1,2}\s\w+\s\w+)/g, "<strong>$1</strong>") // Met en gras les adresses ou dates
      .replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>") // Formatage en gras **Texte**
      .replace(/\*(.*?)\*/g, "<em>$1</em>") // Formatage en italique *Texte*
      .replace(/- (.*?)<br>/g, "• $1<br>"); // Convertir les listes en puces
}

watch(isChatActive, (newVal) => {
     if (newVal) {
          nextTick(() => {
               scrollToBottom()
          })
     }
})


</script>

<template>
     <!-- Bouton flottant -->
     <button v-if="!isOpen" @click="toggleChat" class=" css_chat-open fixed bottom-6 right-6 shadow-lg w-[60px] h-[60px] rounded-[28px] flex items-center justify-center text-white bg-[#007dfc] shadow-[rgba(2,6,16,0.2)_0px_4px_24px] hover:scale-110 focus:scale-110 transition-transform duration-200">
          <svg fill="white" height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true"> <path d="M20 2H4c-1.1 0-2 .9-2 2v18l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2z"></path> <path d="M0 0h24v24H0z" fill="none"></path> </svg>
     </button>

     <!-- Chatbot -->

     <div v-if="isOpen" v-auto-animate class="hhcss_chat fixed bottom-6 right-6 shadow-lg rounded-lg transition-all border border-gray-200 flex flex-col" :class="isMobile ? 'w-full h-full top-0 left-0' : 'w-96 h-[88vh]'">

          <!-- Header -->
          <div class="css_header-bg">
                <div class="css_header-bg  p-4 bg-gradient-to-r from-blue-500 to-indigo-500 text-white flex justify-between items-center rounded-t-lg">
                     <div class="css_header-logo" v-if="!isChatActive">
                       <svg width="189" height="28" viewBox="0 0 189 28" fill="none" xmlns="http://www.w3.org/2000/svg">
                         <path d="M52.47 0C54.54 0 56.23 1.68 56.23 3.76V23.67C56.23 25.74 54.55 27.43 52.47 27.43C50.39 27.43 48.71 25.75 48.71 23.67V3.76C48.71 1.69 50.39 0 52.47 0ZM62.67 0C64.74 0 66.43 1.68 66.43 3.76V23.67C66.43 25.74 64.75 27.43 62.67 27.43C60.59 27.43 58.91 25.75 58.91 23.67V3.76C58.91 1.69 60.59 0 62.67 0ZM42.89 18.78H33.01C33.56 20.58 35.12 21.77 37.36 21.77C39.6 21.77 40.65 20.75 42.39 20.75C43.95 20.75 45.14 21.69 45.14 23.29C45.14 25.99 41.59 27.55 37.12 27.55C30.7 27.55 25.97 22.88 25.97 17.15C25.97 11.42 30.19 6.71 36.74 6.71C43.29 6.71 46.75 11.05 46.75 15.06C46.75 17.27 45.44 18.79 42.91 18.79L42.89 18.78ZM33.13 14.77H39.89C39.34 13.01 38.07 12.23 36.51 12.23C34.86 12.23 33.68 13.21 33.13 14.77ZM79.27 6.7C85.16 6.7 90 11.35 90 17.11C90 22.87 85.16 27.48 79.27 27.48C73.38 27.48 68.54 22.83 68.54 17.11C68.54 11.39 73.38 6.7 79.27 6.7ZM79.27 12.86C77 12.86 75.31 14.7 75.31 17.11C75.31 19.52 76.99 21.31 79.27 21.31C81.55 21.31 83.23 19.47 83.23 17.11C83.23 14.75 81.55 12.86 79.27 12.86ZM19.83 1.09C17.61 1.09 15.8 2.89 15.8 5.12V13.59C14.22 15.07 11.7 14.97 10.69 14.85C10.6 14.84 10.53 14.93 10.56 15.01C10.82 15.69 11.96 17.83 15.8 18.61V23.39C15.8 25.61 17.6 27.42 19.83 27.42C22.06 27.42 23.86 25.62 23.86 23.39V5.12C23.86 2.9 22.06 1.09 19.83 1.09ZM4 13.47C3.99 13.37 4.09 13.3 4.18 13.35C4.72 13.63 6.29 14.37 8.06 14.49V5.12C8.06 2.9 6.26 1.09 4.03 1.09C1.8 1.09 0 2.89 0 5.12V23.4C0 25.62 1.8 27.43 4.03 27.43C6.26 27.43 8.06 25.63 8.06 23.4V18.27C4.76 17.32 4.12 14.33 4 13.47Z" fill="white"/>
                         <path d="M109 9.398C109.954 9.398 110.74 10.208 110.74 11.162V24.398C110.74 25.352 109.954 26.162 109 26.162C108.046 26.162 107.26 25.352 107.26 24.398V19.142H98.98V24.398C98.98 25.352 98.194 26.162 97.24 26.162C96.286 26.162 95.5 25.352 95.5 24.398V11.162C95.5 10.208 96.286 9.398 97.24 9.398C98.194 9.398 98.98 10.208 98.98 11.162V15.986H107.26V11.162C107.26 10.208 108.046 9.398 109 9.398ZM123.46 13.67C124.396 13.67 125.146 14.42 125.146 15.356V24.476C125.146 25.412 124.396 26.162 123.46 26.162C122.542 26.162 121.786 25.412 121.786 24.476V24.218C121.306 25.25 120.262 26.324 118.102 26.324C115.702 26.324 113.746 24.686 113.746 21.758V15.356C113.746 14.42 114.502 13.67 115.42 13.67C116.356 13.67 117.106 14.42 117.106 15.356V20.384C117.106 22.466 118.096 23.168 119.326 23.168C120.526 23.168 121.786 22.466 121.786 20.384V15.356C121.786 14.42 122.542 13.67 123.46 13.67ZM142.812 13.508C145.092 13.508 146.808 15.062 146.808 17.99V24.476C146.808 25.412 146.058 26.162 145.134 26.162C144.204 26.162 143.448 25.412 143.448 24.476V19.448C143.448 17.366 142.578 16.664 141.426 16.664C140.346 16.664 139.164 17.366 139.164 19.448V24.476C139.164 25.412 138.414 26.162 137.49 26.162C136.56 26.162 135.804 25.412 135.804 24.476V19.448C135.804 17.366 134.934 16.664 133.782 16.664C132.702 16.664 131.52 17.366 131.52 19.448V24.476C131.52 25.412 130.77 26.162 129.846 26.162C128.916 26.162 128.16 25.412 128.16 24.476V15.356C128.16 14.42 128.916 13.67 129.846 13.67C130.77 13.67 131.52 14.42 131.52 15.356V15.614C132 14.582 133.086 13.508 135.168 13.508C136.812 13.508 138.162 14.312 138.786 15.86C139.56 14.42 140.898 13.508 142.812 13.508ZM155.36 13.592C158.906 13.592 160.688 14.984 160.688 18.452V24.476C160.688 25.412 159.938 26.162 159.014 26.162C158.084 26.162 157.328 25.412 157.328 24.476V24.296C156.524 25.646 155.198 26.324 153.59 26.324C151.31 26.324 149.384 24.992 149.384 22.508C149.384 19.994 151.568 18.584 154.532 18.584C155.432 18.584 156.32 18.734 157.316 19.058V18.746C157.316 17.306 156.608 16.514 154.748 16.514C153.788 16.514 153.068 16.748 152.546 16.934C152.108 17.102 151.796 17.216 151.34 17.216C150.734 17.216 150.098 16.736 150.098 15.908C150.098 15.242 150.5 14.708 151.232 14.39C152.204 13.982 153.59 13.592 155.36 13.592ZM154.448 23.828C155.984 23.828 157.274 22.754 157.322 21.056V20.966C156.5 20.618 155.69 20.474 154.988 20.474C153.386 20.474 152.558 21.068 152.558 22.202C152.558 23.318 153.32 23.828 154.448 23.828ZM170.879 13.508C173.279 13.508 175.355 15.062 175.355 17.99V24.476C175.355 25.412 174.605 26.162 173.681 26.162C172.751 26.162 171.995 25.412 171.995 24.476V19.448C171.995 17.366 170.927 16.664 169.655 16.664C168.455 16.664 167.075 17.366 167.075 19.448V24.476C167.075 25.412 166.325 26.162 165.401 26.162C164.471 26.162 163.715 25.412 163.715 24.476V15.356C163.715 14.42 164.471 13.67 165.401 13.67C166.325 13.67 167.075 14.42 167.075 15.356V15.614C167.555 14.582 168.719 13.508 170.879 13.508ZM181.303 17.156C181.303 18.848 188.023 17.708 188.023 22.334C188.023 25.064 185.695 26.324 182.521 26.324C181.063 26.324 179.635 26.072 178.669 25.688C178.093 25.466 177.709 24.992 177.709 24.38C177.709 23.6 178.261 23 179.113 23C180.103 23 180.961 23.624 182.611 23.624C183.775 23.624 184.687 23.402 184.687 22.592C184.687 20.702 178.027 21.884 178.027 17.396C178.027 14.738 180.391 13.508 183.211 13.508C184.519 13.508 185.797 13.742 186.691 14.108C187.333 14.366 187.693 14.798 187.693 15.494C187.693 16.232 187.135 16.808 186.301 16.808C185.401 16.808 184.597 16.166 183.163 16.166C181.735 16.166 181.303 16.634 181.303 17.156Z" fill="white"/>
                       </svg>
                     </div >

                      <div v-if="isChatActive">
                            <button @click="isChatActive = false" class="relative group p-2 rounded-full transition hover:bg-[#00245c29] w-10 h-10 flex items-center justify-center">
                              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="directional-icon"><path d="M16.6312 17.7375L14.8687 19.5L7.36865 12L14.8687 4.5L16.6312 6.2625L10.9062 12L16.6312 17.7375Z"></path></svg>
                            </button>
                      </div>
                      <div class="css_header-logo2" v-if="isChatActive">
                              <div>
                                     <svg width="24" height="27" viewBox="0 0 24 27" fill="none" xmlns="http://www.w3.org/2000/svg">
                                     <path d="M19.83 0C17.61 0 15.8 1.8 15.8 4.03V12.5C14.22 13.98 11.7 13.88 10.69 13.76C10.6 13.75 10.53 13.84 10.56 13.92C10.82 14.6 11.96 16.74 15.8 17.52V22.3C15.8 24.52 17.6 26.33 19.83 26.33C22.06 26.33 23.86 24.53 23.86 22.3V4.03C23.86 1.81 22.06 0 19.83 0ZM4 12.38C3.99 12.28 4.09 12.21 4.18 12.26C4.72 12.54 6.29 13.28 8.06 13.4V4.03C8.06 1.81 6.26 0 4.03 0C1.8 0 0 1.8 0 4.03V22.31C0 24.53 1.8 26.34 4.03 26.34C6.26 26.34 8.06 24.54 8.06  22.31V17.18C4.76 16.23 4.12 13.24 4 12.38Z" fill="white"/>
                                     </svg>
                              </div>
                              <div>Bonjour ! 👋</div>
                      </div>
                      <div class="flex space-x-2">
                          <!-- Open options -->
                              <button @click="toggleOptions" class="relative group p-2 rounded-full transition hover:bg-[#00245c29] testddd w-10 h-10 flex items-center justify-center">
                                <span class="absolute -top-6 left-1/2 transform -translate-x-1/2 bg-gray-700 text-white text-xs rounded px-2 py-1 opacity-0 group-hover:opacity-100 transition">Open options</span>
                               <svg fill="white" height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"> <path d="M12 8c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zm0 2c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0 6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2z"></path> </svg>
                              </button>

                          <!-- Minimize -->
                              <button @click="toggleChat" class="relative group p-2 rounded-full transition hover:bg-[#00245c29] w-10 h-10 flex items-center justify-center">
                                <span class="absolute -top-6 left-1/2 transform -translate-x-1/2 bg-gray-700 text-white text-xs rounded px-2 py-1 opacity-0 group-hover:opacity-100 transition">Minimize</span>
                               <svg fill="#fff" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"> <path d="M17.7375 7.36914L19.5 9.13164L12 16.6316L4.5 9.13164L6.2625 7.36914L12 13.0941L17.7375 7.36914Z"></path> </svg>
                              </button>
                      </div>
                </div>

                 <!-- Message de bienvenue -->
                 <div class="css_chat-container" v-if="!isChatActive">
                       <p class="css_chat-container-st">Bonjour ! 👋</p>
                       <p>Bienvenue sur le site de l'Office de Tourisme, Avez vous besoin d’aider ?</p>
                 </div>
                 <div class="css_chat-container" v-if="isChatActive">
                       <p>Obtenez des réponses rapides et personnalisées en un instant !</p>
                 </div>
          </div>

           <div class="hhcss_headerWave">
                 <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 372 15">
                      <path d="M349.8 1.4C334.5.4 318.5 0 302 0h-2.5c-9.1 0-18.4.1-27.8.4-34.5 1-68.3 3-102.3 4.7-14 .5-28 1.2-41.5 1.6C84 7.7 41.6 5.3 0 2.2v8.4c41.6 3 84 5.3 128.2 4.1 13.5-.4 27.5-1.1 41.5-1.6 33.9-1.7 67.8-3.6 102.3-4.7 9.4-.3 18.7-.4 27.8-.4h2.5c16.5 0 32.4.4 47.8 1.4 8.4.3 15.6.7 22 1.2V2.2c-6.5-.5-13.8-.5-22.3-.8z" fill="#fff">
                      </path>
                 </svg>
           </div>

          <!-- Home (Accueil) -->
           <div v-if="!isChatActive" class=" css_home p-4 space-y-4">

               <!-- Questions suggérées -->
                <div class="css_accueil p-3 max-h-50 overflow-y-auto">
                       <div>
                            <button v-for="(question, index) in suggestedQuestions" :key="index" @click="sendSuggestedMessage(question)" class="css_accueilbtn text-left p-3 hover:bg-gray-200 transition">{{ question }}
                                  <div class="css_iconstartchat">
                                        <svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
                                          <path d="M7.5 6.175L8.675 5L13.675 10L8.675 15L7.5 13.825L11.3167 10L7.5 6.175Z" fill="#494949"/>
                                        </svg>
                                  </div>
                            </button>
                       </div>

                </div>

                <div class="css_but-pna">

                            <!-- Bouton Parlez à notre assistan -->
                      <button class="css_but-pnat" @click="isChatActive = true">
                             <div>
                                    Parlez à notre assistant
                                    <br>
                                    <small>Obtenez des réponses rapides et personnalisées en un instant !</small>
                             </div>
                             <div class="css_iconstartchat">
                                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="21" viewBox="0 0 20 21" fill="none">
                                      <path d="M4.82004 9.47833L3.34552 6.5293C2.2362 4.31066 1.68155 3.20135 2.1912 2.69169C2.70085 2.18204 3.81017 2.7367 6.0288 3.84601L15.7178 8.69056C17.2789 9.4711 18.0595 9.86138 18.0595 10.4794C18.0595 11.0974 17.2789 11.4877 15.7179 12.2683L6.02881 17.1128C3.81017 18.2221 2.70085 18.7768 2.1912 18.2671C1.68155 17.7575 2.2362 16.6482 3.34552 14.4295L4.82112 11.4783H10.5307C11.0829 11.4783 11.5307 11.0306 11.5307 10.4783C11.5307 9.92605 11.0829 9.47833 10.5307 9.47833H4.82004Z" fill="#3866FB"/>
                                    </svg>
                             </div>
                      </button>
                </div>

                <!-- Powered by HelloHumans -->
                 <div class=" css_pbh text-center text-sm text-gray-500">Powered by HelloHumans</div>

                 <hr class="hr">

               <!-- Navigation en bas -->
                <div class="flex bg-white">
                      <!-- Home -->
                      <button @click="isChatActive = false" class="flex-1 flex flex-col items-center justify-center"><svg width="28" height="28" viewBox="0 0 28 28" fill="none" xmlns="http://www.w3.org/2000/svg"> <g> <path d="M4.66663 24.5V10.5L14 3.5L23.3333 10.5V24.5H16.3333V16.3333H11.6666V24.5H4.66663Z" fill="#0566ff"></path> </g> </svg><span class="text-sm text-gray-700">Home</span>
                      </button>

                      <!-- Chat -->
                      <button @click="isChatActive = true" class="flex-1 flex flex-col items-center justify-center"><svg width="28" height="28" viewBox="0 0 28 28" fill="none" xmlns="http://www.w3.org/2000/svg"> <g> <path d="M2.91663 23.9613V4.44205C2.91663 3.85269 3.12079 3.35384 3.52913 2.94551C3.93746 2.53717 4.43631 2.33301 5.02567 2.33301H22.9743C23.5636 2.33301 24.0625 2.53717 24.4708 2.94551C24.8791 3.35384 25.0833 3.85269 25.0833 4.44205V17.724C25.0833 18.3133 24.8791 18.8122 24.4708 19.2205C24.0625 19.6288 23.5636 19.833 22.9743 19.833H7.04488L2.91663 23.9613ZM6.29996 18.083H22.9743C23.0641 18.083 23.1463 18.0456 23.221 17.9707C23.2959 17.896 23.3333 17.8138 23.3333 17.724V4.44205C23.3333 4.35222 23.2959 4.26997 23.221 4.1953C23.1463 4.12044 23.0641 4.08301 22.9743 4.08301H5.02567C4.93583 4.08301 4.85358 4.12044 4.77892 4.1953C4.70406 4.26997 4.66663 4.35222 4.66663 4.44205V19.6985L6.29996 18.083Z" fill="#4C596B"></path> </g> </svg><span class="text-sm text-gray-700">Chat</span>
                      </button>
                </div>
           </div>


          <!-- Messages -->
          <div ref="chatContainer"  v-auto-animate v-if="isChatActive" class="flex-1 overflow-y-auto p-4 space-y-3 bg-gray-50">
               <div v-for="(msg, index) in messages" :key="index" class="css_messages flex items-center"  :class=" msg.sender === 'user' ? 'justify-end' : 'justify-start'">
                    <p v-html="formatMessage(msg.text)" :class="msg.sender === 'user' ? 'bg-blue-500 text-white' : 'bg-white text-gray-800 border border-gray-200'" class="inline-block rounded-2xl px-4 py-3  max-w-xs leading-relaxed"></p>
               </div>
               <div v-if="isLoading" class="text-left animate-pulse">
                    <p class="bg-gray-300 text-gray-700 inline-block rounded-lg px-3 py-2 my-1 max-w-xs">Chatbot est en train d'écrire...</p>
               </div>
          </div>

          <div v-if="isChatActive">
               <!-- Ligne de séparation -->
               <div class="border-t border-gray-80"></div>

               <!-- Saisie utilisateur -->
               <div class="p-4 bg-white flex items-center">
                    <input v-model="message" class="w-full rounded-md focus:outline-none" placeholder="Écrivez un message..." @keyup.enter="sendMessage" />
                    <button @click="sendMessage" class="relative group p-2 rounded-full transition hover:bg-[#00245c29] w-10 h-10 flex items-center justify-center ml-2"><span class="absolute -top-6 left-1/2 transform -translate-x-1/2 bg-gray-700 text-white text-xs rounded px-2 py-1 opacity-0 group-hover:opacity-100 transition">Envoyer</span><svg width="20" height="21" viewBox="0 0 20 21" fill="none" xmlns="http://www.w3.org/2000/svg"> <g> <path d="M4.8198 9.47833L3.34528 6.5293C2.23596 4.31066 1.68131 3.20135 2.19096 2.69169C2.70061 2.18204 3.80993 2.7367 6.02856 3.84601L15.7176 8.69056C17.2787 9.4711 18.0593 9.86138 18.0593 10.4794C18.0593 11.0974 17.2787 11.4877 15.7177 12.2683L6.02857 17.1128C3.80993 18.2221 2.70061 18.7768 2.19096 18.2671C1.68131 17.7575 2.23596 16.6482 3.34528 14.4295L4.82088 11.4783H10.5305C11.0827 11.4783 11.5305 11.0306 11.5305 10.4783C11.5305 9.92605 11.0827 9.47833 10.5305 9.47833H4.8198Z" fill="var(--custom-action-color, #0566ff)"></path> </g></svg>
                    </button>
               </div>
          </div>
          <!-- Options -->
          <div v-if="showOptions" class="absolute top-14 right-4 bg-white shadow-lg rounded-md p-3 border border-gray-200 w-56">
               <button @click="toggleNotifications" class="w-full bg-white shadow rounded-md p-2 flex items-center space-x-2 hover:bg-gray-100 transition"><svg fill="black" height="24" viewBox="0 0 24 24" width="24"> <path d="M0 0h24v24H0z" fill="none"></path> <path d="M20 18.69L7.84 6.14 5.27 3.49 4 4.76l2.8 2.8v.01c-.52.99-.8 2.16-.8 3.42v5l-2 2v1h13.73l2 2L21 19.72l-1-1.03zM12 22c1.11 0 2-.89 2-2h-4c0 1.11.89 2 2 2zm6-7.32V11c0-3.08-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68c-.15.03-.29.08-.42.12-.1.03-.2.07-.3.11h-.01c-.01 0-.01 0-.02.01-.23.09-.46.2-.68.31 0 0-.01 0-.01.01L18 14.68z"></path></svg><span>{{ notificationsEnabled ? "Turn off notifications" : "Turn on notifications" }}</span>
               </button>
          </div>

     </div>
</template>
