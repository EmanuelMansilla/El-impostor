<template>
  <div id="app">
    <div class="banner">
      <h1>EL IMPOSTOR</h1>
    </div>
    <div v-if="!gameStarted && !isRevealing">
      <div class="setup">
        <label for="numPlayers">Número de jugadores:</label>
        <input v-model.number="numPlayers" type="number" min="2" max="10" id="numPlayers" />
        <br />
        <label for="category">Categoría de personajes:</label>
        <select v-model="selectedCategory" id="category">
          <option value="">Selecciona una categoría</option>
          <option value="football">Jugadores de Fútbol</option>
          <option value="movies">Personajes de Películas</option>
          <option value="cartoons">Personajes de Dibujos Animados</option>
          <option value="anime">Personajes de Anime</option>
        </select>
        <br />
        <button @click="startGame" :disabled="!canStart">¿Qué personaje serás?</button>
      </div>
    </div>
    <div v-if="isRevealing">
      <div class="reveal">
        <div v-if="readyForReveal">
          <p>Presiona siguiente para revelar al jugador {{ currentReveal + 1 }}</p>
          <button @click="startCountdown">Siguiente</button>
        </div>
        <div v-if="isCountingDown">
          <p>¡Y tu personaje es!</p>
          <p style="font-size: 3em; color: #ff0000;">{{ countdown }}</p>
        </div>
        <div v-if="showRole">
          <p style="font-size: 2em; color: #ff0000;">{{ revealMessages[currentReveal] }}</p>
        </div>
      </div>
    </div>
    <div v-if="gameStarted">
      <div class="game">
        <h2>¡Que comience el juego!</h2>
        <button @click="resetGame">Nueva Partida</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const numPlayers = ref(4)
const selectedCategory = ref('')
const character = ref('')
const gameStarted = ref(false)
const isImpostor = ref(false)
const isRevealing = ref(false)
const currentReveal = ref(0)
const revealMessages = ref<string[]>([])
const isCountingDown = ref(false)
const showRole = ref(false)
const countdown = ref(5)
const countdownTimer = ref<NodeJS.Timeout | null>(null)

const readyForReveal = computed(() => !isCountingDown.value && !showRole.value && isRevealing.value)

const characters = {
  football: [
    'Lionel Messi', 'Cristiano Ronaldo', 'Neymar Jr', 'Kylian Mbappé', 'Erling Haaland',
    'Kevin De Bruyne', 'Mohamed Salah', 'Son Heung-min', 'Bruno Fernandes', 'Harry Kane',
    'Vinicius Jr', 'Pedri', 'Gavi', 'Jude Bellingham', 'Jamal Musiala',
    'Phil Foden', 'Bukayo Saka', 'Marcus Rashford', 'Raheem Sterling', 'Trent Alexander-Arnold',
    'Alisson Becker', 'Ederson', 'Manuel Neuer', 'Gianluigi Donnarumma', 'Thibaut Courtois',
    'Sergio Ramos', 'Virgil van Dijk', 'Marquinhos', 'João Cancelo', 'Achraf Hakimi'
  ],
  movies: [
    'Luke Skywalker', 'Darth Vader', 'Han Solo', 'Princess Leia', 'Yoda',
    'Frodo Baggins', 'Gandalf', 'Aragorn', 'Legolas', 'Gimli',
    'Tony Stark', 'Captain America', 'Thor', 'Black Widow', 'Hulk',
    'Wonder Woman', 'Superman', 'Batman', 'Joker', 'Harley Quinn',
    'James Bond', 'Indiana Jones', 'Jack Sparrow', 'Elsa', 'Anna',
    'Simba', 'Mufasa', 'Scar', 'Timon', 'Pumbaa',
    'Harry Potter', 'Hermione Granger', 'Ron Weasley', 'Voldemort', 'Dumbledore'
  ],
  cartoons: [
    'Mickey Mouse', 'Donald Duck', 'Goofy', 'Minnie Mouse', 'Pluto',
    'Bugs Bunny', 'Daffy Duck', 'Tweety', 'Sylvester', 'Road Runner',
    'Tom', 'Jerry', 'Scooby-Doo', 'Shaggy', 'Velma',
    'SpongeBob SquarePants', 'Patrick Star', 'Squidward', 'Mr. Krabs', 'Plankton',
    'Homer Simpson', 'Marge Simpson', 'Bart Simpson', 'Lisa Simpson', 'Maggie Simpson',
    'Peter Griffin', 'Lois Griffin', 'Stewie Griffin', 'Brian Griffin', 'Meg Griffin',
    'Rick Sanchez', 'Morty Smith', 'Summer Smith', 'Beth Smith', 'Jerry Smith'
  ],
  anime: [
    'Naruto Uzumaki', 'Sasuke Uchiha', 'Sakura Haruno', 'Kakashi Hatake', 'Itachi Uchiha',
    'Monkey D. Luffy', 'Roronoa Zoro', 'Nami', 'Usopp', 'Sanji',
    'Ichigo Kurosaki', 'Rukia Kuchiki', 'Orihime Inoue', 'Uryu Ishida', 'Chad',
    'Edward Elric', 'Alphonse Elric', 'Winry Rockbell', 'Roy Mustang', 'Riza Hawkeye',
    'Goku', 'Vegeta', 'Piccolo', 'Gohan', 'Krillin',
    'Light Yagami', 'L Lawliet', 'Misa Amane', 'Ryuk', 'Near',
    'Tanjiro Kamado', 'Nezuko Kamado', 'Zenitsu Agatsuma', 'Inosuke Hashibira', 'Kanao Tsuyuri'
  ]
}

const canStart = computed(() => numPlayers.value >= 2 && selectedCategory.value !== '')

const startGame = () => {
  // Seleccionar personaje aleatorio único para todos los jugadores
  const categoryChars = characters[selectedCategory.value as keyof typeof characters]
  const randomChar = categoryChars[Math.floor(Math.random() * categoryChars.length)]
  character.value = randomChar

  // Asignar impostor aleatoriamente
  const impostorIndex = Math.floor(Math.random() * numPlayers.value)
  isImpostor.value = impostorIndex === 0 // Usuario es impostor si índice 0

  // Crear mensajes de revelación
  revealMessages.value = []
  for (let i = 1; i <= numPlayers.value; i++) {
    if (i - 1 === impostorIndex) {
      // Este jugador es el impostor
      revealMessages.value.push(`Jugador ${i}: Eres el Impostor`)
    } else {
      // Todos los demás tienen el mismo personaje
      revealMessages.value.push(`Jugador ${i}: Eres ${character.value}`)
    }
  }

  isRevealing.value = true
  currentReveal.value = 0
}

const startCountdown = () => {
  isCountingDown.value = true
  countdown.value = 5

  countdownTimer.value = setInterval(() => {
    countdown.value--

    if (countdown.value <= 0) {
      clearInterval(countdownTimer.value!)
      countdownTimer.value = null
      isCountingDown.value = false
      showRole.value = true

      // Después de 5 segundos, ocultar el rol y avanzar
      setTimeout(() => {
        showRole.value = false
        if (currentReveal.value < revealMessages.value.length - 1) {
          currentReveal.value++
        } else {
          isRevealing.value = false
          gameStarted.value = true
        }
      }, 5000)
    }
  }, 1000)
}

const nextReveal = () => {
  showRole.value = false
  isCountingDown.value = false
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value)
    countdownTimer.value = null
  }

  if (currentReveal.value < revealMessages.value.length - 1) {
    currentReveal.value++
  } else {
    isRevealing.value = false
    gameStarted.value = true
  }
}

const resetGame = () => {
  gameStarted.value = false
  isImpostor.value = false
  selectedCategory.value = ''
  character.value = ''
  isRevealing.value = false
  currentReveal.value = 0
  revealMessages.value = []
  isCountingDown.value = false
  showRole.value = false
  countdown.value = 5
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value)
    countdownTimer.value = null
  }
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
  background-color: #333;
  min-height: 100vh;
}

.banner {
  background-color: #000;
  background-image: url('https://images.unsplash.com/photo-1578662996442-48f60103fc96?ixlib=rb-4.0.3&auto=format&fit=crop&w=1350&q=80');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  padding: 40px 20px;
  margin-bottom: 40px;
  border-bottom: 3px solid #ff0000;
  position: relative;
}

.banner::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 1;
}

.banner h1 {
  position: relative;
  z-index: 2;
  color: #ff0000;
  font-size: 4rem;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(255, 0, 0, 0.5);
  margin: 0;
  letter-spacing: 0.2em;
}

.setup, .game, .reveal {
  margin: 20px auto;
  max-width: 400px;
  padding: 20px;
  border: 2px solid #ff0000;
  border-radius: 8px;
  background-color: #f9f9f9;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.reveal p {
  font-size: 1.2em;
  margin-bottom: 20px;
}

input, select {
  margin: 10px 0;
  padding: 8px;
  width: 100%;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #ff0000;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  font-weight: bold;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #cc0000;
}
</style>
