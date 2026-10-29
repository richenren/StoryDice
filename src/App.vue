<script setup>
import { computed, ref, watch } from 'vue'
import { iconLibrary, blindboxCategories, starterLines } from './data/cards.js'

const mode = ref('dice')
const diceCount = 6
const lockedIds = ref([])
const diceCards = ref([])
const blindbox = ref({})
const starterIndex = ref(0)
const storyTitle = ref('')
const storyText = ref('')
const savedStories = ref([])
const showSaved = ref(false)
const notice = ref('')

const tones = {
  peach: '#ffe0cc', rose: '#ffdce5', blue: '#dcecff',
  mint: '#d9f4e8', yellow: '#fff0b8', lavender: '#e9e1ff', lilac: '#e8ddff'
}
const iconColors = {
  kid: '#e98c69', grandma: '#c97870', robot: '#5d9ad6', dragon: '#50a88c',
  cat: '#a78bd6', wizard: '#8a79c7', school: '#e4ae42', forest: '#4e9b73',
  space: '#6e86c8', island: '#e4a76c', castle: '#cf7b8d', kitchen: '#d7a13e',
  treasure: '#d9a52f', ball: '#54a780', rain: '#5d9ed0', fly: '#8b83d8',
  birthday: '#d77b9a', lost: '#dc946c', morning: '#e9b449', night: '#6179ba',
  happy: '#e7b43b', scared: '#9885c8', curious: '#50a890', silly: '#e99570'
}

function sampleCards(count, excluded = []) {
  const available = iconLibrary.filter(item => !excluded.includes(item.id))
  const pool = [...available]
  const result = []
  while (result.length < count && pool.length) {
    const index = Math.floor(Math.random() * pool.length)
    result.push(pool.splice(index, 1)[0])
  }
  return result
}
function rollDice() {
  const fixed = diceCards.value.filter(card => lockedIds.value.includes(card.id))
  const next = sampleCards(diceCount - fixed.length, fixed.map(card => card.id))
  diceCards.value = [...fixed, ...next]
}
function toggleLock(id) {
  lockedIds.value = lockedIds.value.includes(id)
    ? lockedIds.value.filter(item => item !== id)
    : [...lockedIds.value, id]
}
function rollBox(categoryId) {
  const category = blindboxCategories.find(item => item.id === categoryId)
  const current = blindbox.value[categoryId]
  const options = category.prompts.filter(item => item !== current)
  blindbox.value[categoryId] = options[Math.floor(Math.random() * options.length)] || current
}
function rollAllBox() {
  blindboxCategories.forEach(category => rollBox(category.id))
}
function randomStarter() {
  starterIndex.value = (starterIndex.value + 1) % starterLines.length
}
function startDraft() {
  const prompts = mode.value === 'dice'
    ? diceCards.value.map(item => item.label).join('、')
    : blindboxCategories.map(category => blindbox.value[category.id]).filter(Boolean).join('、')
  storyTitle.value = storyTitle.value || '我的奇妙故事'
  storyText.value = storyText.value ? storyText.value + '\n' : ''
  storyText.value += starterLines[starterIndex.value] + '\n今天的灵感：' + prompts + '。\n'
  showSaved.value = false
  document.querySelector('#story-editor')?.scrollIntoView({ behavior: 'smooth', block: 'center' })
}
function loadStories() {
  try { savedStories.value = JSON.parse(localStorage.getItem('storydice-stories') || '[]') }
  catch { savedStories.value = [] }
}
function persistStories() {
  localStorage.setItem('storydice-stories', JSON.stringify(savedStories.value))
}
function saveStory() {
  const text = storyText.value.trim()
  if (!text) { notice.value = '先写一点故事内容，再收藏吧！'; return }
  const story = {
    id: Date.now(),
    title: storyTitle.value.trim() || '没有名字的故事',
    text,
    createdAt: new Date().toLocaleDateString('zh-CN')
  }
  savedStories.value = [story, ...savedStories.value]
  persistStories()
  notice.value = '故事收藏成功！'
}
function openStory(story) {
  storyTitle.value = story.title
  storyText.value = story.text
  showSaved.value = false
}
function deleteStory(id) {
  savedStories.value = savedStories.value.filter(story => story.id !== id)
  persistStories()
}
function setMode(nextMode) {
  mode.value = nextMode
  showSaved.value = false
}
watch(notice, value => {
  if (value) setTimeout(() => { if (notice.value === value) notice.value = '' }, 2600)
})
rollDice()
rollAllBox()
loadStories()
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <a class="brand" href="#" @click.prevent="setMode('dice')" aria-label="回到故事星球首页">
        <span class="brand-mark"><span></span><span></span><span></span></span>
        <span><b>故事星球</b><small>STORY DICE</small></span>
      </a>
      <button class="saved-link" @click="showSaved = !showSaved">
        <span class="saved-icon">♡</span> 我的故事 <span class="saved-count">{{ savedStories.length }}</span>
      </button>
    </header>

    <main>
      <section class="hero">
        <div class="hero-copy">
          <div class="eyebrow"><span class="sparkle">✦</span> 给想象力一点小小的魔法</div>
          <h1>今天，来编一个<br><span>只属于你的故事！</span></h1>
          <p>抽一抽灵感，把奇妙的图案变成冒险。没有标准答案，每个故事都很棒。</p>
          <div class="hero-note"><span class="note-star">✦</span> 适合和爸爸妈妈一起玩，也可以自己当故事大王。</div>
        </div>
        <div class="hero-art" aria-hidden="true">
          <div class="sun-shape"></div><div class="cloud cloud-one"></div><div class="cloud cloud-two"></div>
          <div class="rainbow"><i></i><i></i><i></i></div>
          <div class="floating-card card-one"><svg viewBox="0 0 80 80"><path d="M18 58 L24 30 L40 17 L56 30 L62 58Z" fill="#e9a66c" stroke="#6e5949" stroke-width="2.5" stroke-linejoin="round"/><path d="M32 58V42H48V58" fill="#fff0bd" stroke="#6e5949" stroke-width="2.5"/><path d="M20 32L40 42L60 32" fill="none" stroke="#6e5949" stroke-width="2.5"/></svg></div>
          <div class="floating-card card-two"><svg viewBox="0 0 80 80"><circle cx="40" cy="40" r="22" fill="#ffcf5d" stroke="#7e6545" stroke-width="2.5"/><path d="M40 12V5M40 75V68M12 40H5M75 40H68M20 20L14 14M60 60L66 66M60 20L66 14M20 60L14 66" stroke="#7e6545" stroke-width="3" stroke-linecap="round"/><circle cx="33" cy="37" r="2" fill="#5b554e"/><circle cx="47" cy="37" r="2" fill="#5b554e"/><path d="M32 46Q40 53 48 46" fill="none" stroke="#5b554e" stroke-width="2.5" stroke-linecap="round"/></svg></div>
          <div class="hero-book"><div class="book-spine"></div><div class="book-page"><span>从前……</span><div class="book-stars">✦ · ✧</div><div class="book-hill"></div></div></div>
          <span class="hero-doodle doodle-a">✦</span><span class="hero-doodle doodle-b">✧</span><span class="hero-doodle doodle-c">✦</span>
        </div>
      </section>

      <div class="mode-tabs" role="tablist" aria-label="选择故事玩法">
        <button :class="{ active: mode === 'dice' && !showSaved }" @click="setMode('dice')"><span class="tab-emoji">🎲</span><span><b>故事骰子</b><small>掷出灵感，随心开讲</small></span></button>
        <button :class="{ active: mode === 'blindbox' && !showSaved }" @click="setMode('blindbox')"><span class="tab-emoji">🎁</span><span><b>故事盲盒</b><small>打开惊喜，拼出冒险</small></span></button>
      </div>

      <section v-if="showSaved" class="workspace saved-workspace">
        <div class="section-heading"><div><span class="section-kicker">MY STORY SHELF</span><h2>我的故事收藏</h2><p>把每一次奇思妙想，好好收藏起来。</p></div></div>
        <div v-if="savedStories.length" class="saved-grid">
          <article v-for="story in savedStories" :key="story.id" class="saved-card">
            <div class="saved-card-art"><span>✦</span><span>故事收藏</span><span>✧</span></div>
            <div class="saved-card-body"><small>{{ story.createdAt }}</small><h3>{{ story.title }}</h3><p>{{ story.text }}</p><div class="saved-actions"><button class="button-soft" @click="openStory(story)">继续编辑</button><button class="delete-button" @click="deleteStory(story.id)">删除</button></div></div>
          </article>
        </div>
        <div v-else class="empty-state"><div class="empty-illustration">✧</div><h3>这里还空空的</h3><p>写下第一个故事，把它收藏进你的故事书吧。</p><button class="button-primary" @click="showSaved = false">开始创作</button></div>
      </section>

      <section v-else-if="mode === 'dice'" class="workspace">
        <div class="section-heading">
          <div><span class="section-kicker">01 · ROLL YOUR IDEAS</span><h2>故事骰子</h2><p>看看今天会遇见哪些奇妙角色和事情？</p></div>
          <div class="roll-tip"><span>☝</span> 点一下图案可以把它锁住</div>
        </div>
        <div class="dice-board">
          <button v-for="(card, index) in diceCards" :key="card.id" class="picture-card" :class="['tone-' + card.tone, { locked: lockedIds.includes(card.id) }]" @click="toggleLock(card.id)" :aria-pressed="lockedIds.includes(card.id)">
            <span class="card-number">{{ String(index + 1).padStart(2, '0') }}</span>
            <span v-if="lockedIds.includes(card.id)" class="lock-badge">锁住啦</span>
            <svg class="picture-svg" viewBox="0 0 120 108" role="img" :aria-label="card.label">
              <g v-if="['kid','grandma','wizard'].includes(card.id)">
                <circle cx="60" cy="35" r="22" fill="#f5c89f" stroke="#6d5a4b" stroke-width="2.4"/>
                <path v-if="card.id !== 'grandma'" d="M38 31Q39 9 61 11Q82 12 82 31L75 24L44 25Z" fill="#6b5144"/>
                <path v-else d="M39 33Q31 8 51 13Q63 2 77 15Q91 28 81 42L73 26L45 25Z" fill="#f5f0e8" stroke="#8a7c6e" stroke-width="2"/>
                <circle cx="52" cy="36" r="2.5" fill="#57483f"/><circle cx="68" cy="36" r="2.5" fill="#57483f"/>
                <path d="M53 46Q60 52 68 45" fill="none" stroke="#a95e55" stroke-width="2.5" stroke-linecap="round"/>
                <path d="M35 96Q36 63 60 62Q84 63 86 96Z" :fill="iconColors[card.id]" stroke="#6d5a4b" stroke-width="2.4"/>
                <path v-if="card.id === 'wizard'" d="M35 20L51 -2L73 20Z" fill="#8c79c8" stroke="#6d5a4b" stroke-width="2.4"/>
              </g>
              <g v-else-if="card.id === 'robot'">
                <rect x="32" y="22" width="56" height="52" rx="12" fill="#a7d8ee" stroke="#4e7186" stroke-width="2.8"/><path d="M60 22V12M54 12H66" stroke="#4e7186" stroke-width="3" stroke-linecap="round"/><circle cx="47" cy="43" r="5" fill="#4e7186"/><circle cx="73" cy="43" r="5" fill="#4e7186"/><path d="M48 58H72" stroke="#4e7186" stroke-width="3" stroke-linecap="round"/><path d="M32 42H21M88 42H99M44 74V91M76 74V91" stroke="#4e7186" stroke-width="6" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'dragon'">
                <path d="M27 78Q18 50 39 31L30 16L49 25Q66 15 78 32L94 26L88 44Q100 67 78 81Z" fill="#7bcaa9" stroke="#427c68" stroke-width="2.5" stroke-linejoin="round"/><circle cx="48" cy="45" r="3" fill="#3d5b4d"/><circle cx="73" cy="45" r="3" fill="#3d5b4d"/><path d="M49 59Q61 69 74 58" fill="none" stroke="#427c68" stroke-width="3" stroke-linecap="round"/><path d="M40 30L37 7L57 25M76 31L92 10L88 39" fill="#b4e6c9" stroke="#427c68" stroke-width="2.5" stroke-linejoin="round"/>
              </g>
              <g v-else-if="card.id === 'cat'">
                <path d="M28 40L27 15L48 29Q60 24 72 29L94 15L91 43Q101 70 78 83Q60 94 42 82Q18 68 28 40Z" fill="#d4c5f3" stroke="#75649a" stroke-width="2.5" stroke-linejoin="round"/><circle cx="47" cy="51" r="3" fill="#544b64"/><circle cx="72" cy="51" r="3" fill="#544b64"/><path d="M55 61L61 66L67 61M61 66V70" fill="none" stroke="#75649a" stroke-width="2.5" stroke-linecap="round"/><path d="M42 64L19 60M42 70L19 76M79 64L101 60M79 70L101 76" stroke="#75649a" stroke-width="2" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'school'">
                <path d="M20 42L60 14L100 42Z" fill="#f4c76d" stroke="#8a6d49" stroke-width="2.5" stroke-linejoin="round"/><rect x="28" y="42" width="64" height="48" rx="3" fill="#fff0bf" stroke="#8a6d49" stroke-width="2.5"/><rect x="51" y="62" width="18" height="28" fill="#d9a45a" stroke="#8a6d49" stroke-width="2"/><rect x="35" y="51" width="10" height="10" fill="#9ed5e8"/><rect x="75" y="51" width="10" height="10" fill="#9ed5e8"/><path d="M60 14V3" stroke="#8a6d49" stroke-width="3"/>
              </g>
              <g v-else-if="card.id === 'forest'">
                <path d="M17 65L39 26L61 65Z" fill="#80bd8d" stroke="#4f8763" stroke-width="2.5" stroke-linejoin="round"/><path d="M48 75L72 19L99 75Z" fill="#5ca77b" stroke="#4f8763" stroke-width="2.5" stroke-linejoin="round"/><path d="M39 65V94M72 75V97" stroke="#8c6549" stroke-width="8" stroke-linecap="round"/><path d="M10 97Q35 88 57 98T108 97" fill="none" stroke="#70b887" stroke-width="4" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'space'">
                <circle cx="60" cy="54" r="28" fill="#7f8ed2" stroke="#5967a2" stroke-width="2.5"/><ellipse cx="60" cy="54" rx="46" ry="13" fill="none" stroke="#9b8be1" stroke-width="5" transform="rotate(-20 60 54)"/><circle cx="51" cy="45" r="4" fill="#d9e6ff"/><circle cx="71" cy="62" r="6" fill="#5967a2"/><path d="M20 20L23 27L30 29L23 32L20 39L17 32L10 29L17 27Z" fill="#f1c65c"/><circle cx="96" cy="25" r="3" fill="#f1c65c"/>
              </g>
              <g v-else-if="card.id === 'island'">
                <path d="M13 79Q32 64 60 73Q87 63 108 80L102 91H18Z" fill="#f0c178" stroke="#b78551" stroke-width="2.3"/><path d="M62 76Q60 53 78 35" fill="none" stroke="#8b6b4d" stroke-width="7" stroke-linecap="round"/><path d="M77 38Q51 39 48 20Q72 16 80 33Q86 11 103 20Q101 38 80 41" fill="#72b88d" stroke="#4c8966" stroke-width="2.5"/>
              </g>
              <g v-else-if="card.id === 'castle'">
                <path d="M24 91V42H36V29H48V42H72V29H84V42H96V91Z" fill="#f1b9c6" stroke="#9e6674" stroke-width="2.5"/><path d="M15 43L30 28L45 43M66 43L81 28L98 43" fill="#e996aa" stroke="#9e6674" stroke-width="2.5"/><path d="M51 91V68Q60 54 69 68V91" fill="#fff0c6" stroke="#9e6674" stroke-width="2.5"/>
              </g>
              <g v-else-if="card.id === 'kitchen'">
                <rect x="25" y="21" width="70" height="70" rx="7" fill="#ffe2aa" stroke="#a37b4e" stroke-width="2.5"/><rect x="34" y="30" width="52" height="20" rx="3" fill="#fff6dc" stroke="#a37b4e" stroke-width="2"/><path d="M34 61H86M34 72H86" stroke="#a37b4e" stroke-width="2"/><circle cx="76" cy="58" r="3" fill="#a37b4e"/><circle cx="76" cy="69" r="3" fill="#a37b4e"/>
              </g>
              <g v-else-if="card.id === 'treasure'">
                <path d="M25 42H95V85Q95 91 89 91H31Q25 91 25 85Z" fill="#e9b94f" stroke="#9a7137" stroke-width="2.8"/><path d="M22 42Q22 24 40 24H80Q98 24 98 42Z" fill="#f6d57c" stroke="#9a7137" stroke-width="2.8"/><rect x="53" y="47" width="14" height="17" rx="3" fill="#fff0bc" stroke="#9a7137" stroke-width="2"/><path d="M60 10L64 20L75 21L67 28L69 39L60 33L51 39L53 28L45 21L56 20Z" fill="#f7d66d" stroke="#9a7137" stroke-width="1.8"/>
              </g>
              <g v-else-if="card.id === 'ball'">
                <circle cx="60" cy="54" r="34" fill="#fffdf7" stroke="#5f806f" stroke-width="2.5"/><path d="M60 36L75 47L69 65H51L45 47Z" fill="#5f806f"/><path d="M60 36L58 21M75 47L91 42M69 65L79 79M51 65L41 79M45 47L29 42" stroke="#5f806f" stroke-width="3" stroke-linecap="round"/><path d="M18 91Q60 78 102 91" fill="none" stroke="#72b887" stroke-width="4" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'rain'">
                <path d="M27 51Q13 48 18 35Q22 24 35 28Q43 9 60 22Q72 13 84 26Q102 26 102 42Q102 53 88 54Z" fill="#b8d9f6" stroke="#6b96c0" stroke-width="2.5"/><path d="M35 65L29 77M59 65L53 77M83 65L77 77" stroke="#5d9ed0" stroke-width="5" stroke-linecap="round"/><path d="M35 84L31 92M59 84L55 92M83 84L79 92" stroke="#8bbce6" stroke-width="4" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'fly'">
                <path d="M22 57Q39 26 61 50Q79 29 100 42Q80 61 61 57Q41 79 22 57Z" fill="#d5cafa" stroke="#8c7bc4" stroke-width="2.5" stroke-linejoin="round"/><circle cx="60" cy="53" r="7" fill="#fff3bd" stroke="#8c7bc4" stroke-width="2"/><path d="M14 25L18 34L27 38L18 42L14 51L10 42L1 38L10 34Z" fill="#eec85e"/><path d="M91 73L95 81L103 85L95 89L91 97L87 89L79 85L87 81Z" fill="#eec85e"/>
              </g>
              <g v-else-if="card.id === 'birthday'">
                <path d="M25 54H95V88H25Z" fill="#f1a9bd" stroke="#a5677b" stroke-width="2.5"/><path d="M20 54Q20 44 30 44Q40 44 40 54Q40 44 50 44Q60 44 60 54Q60 44 70 44Q80 44 80 54Q80 44 90 44Q100 44 100 54Z" fill="#fff0c5" stroke="#a5677b" stroke-width="2.5"/><path d="M40 44V27M60 44V22M80 44V27" stroke="#a5677b" stroke-width="3"/><path d="M36 26Q40 17 44 26M56 21Q60 12 64 21M76 26Q80 17 84 26" fill="#f1c65c" stroke="#a5677b" stroke-width="1.8"/>
              </g>
              <g v-else-if="card.id === 'lost'">
                <circle cx="60" cy="50" r="31" fill="#ffdfb6" stroke="#a87e58" stroke-width="2.5"/><text x="60" y="62" font-size="39" font-weight="800" text-anchor="middle" fill="#a87e58">?</text><path d="M21 90Q60 78 99 90" fill="none" stroke="#c49b72" stroke-width="4" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'morning' || card.id === 'happy'">
                <circle cx="60" cy="53" r="25" fill="#ffcf5d" stroke="#a9803f" stroke-width="2.5"/><path d="M60 12V20M60 86V94M19 53H27M93 53H101M31 24L37 30M83 76L89 82M89 24L83 30M37 76L31 82" stroke="#a9803f" stroke-width="4" stroke-linecap="round"/><circle cx="51" cy="49" r="2.5" fill="#6c5841"/><circle cx="69" cy="49" r="2.5" fill="#6c5841"/><path d="M50 60Q60 69 70 60" fill="none" stroke="#6c5841" stroke-width="2.5" stroke-linecap="round"/>
              </g>
              <g v-else-if="card.id === 'night'">
                <path d="M76 20Q47 30 54 55Q60 78 83 75Q66 96 43 83Q17 67 29 41Q43 14 76 20Z" fill="#7887c6" stroke="#5969a4" stroke-width="2.5"/><path d="M88 25L92 34L101 38L92 42L88 51L84 42L75 38L84 34Z" fill="#f4cc64"/><circle cx="37" cy="24" r="3" fill="#f4cc64"/>
              </g>
              <g v-else>
                <circle cx="60" cy="48" r="29" fill="#f6d4ba" stroke="#a77961" stroke-width="2.5"/><circle cx="49" cy="43" r="3" fill="#795c4f"/><circle cx="71" cy="43" r="3" fill="#795c4f"/><path v-if="card.id === 'scared'" d="M51 60Q60 51 69 60" fill="none" stroke="#795c4f" stroke-width="2.8" stroke-linecap="round"/><path v-else-if="card.id === 'silly'" d="M50 57Q60 72 72 55" fill="none" stroke="#795c4f" stroke-width="3" stroke-linecap="round"/><path v-else d="M51 57Q60 64 69 57" fill="none" stroke="#795c4f" stroke-width="2.8" stroke-linecap="round"/><circle cx="40" cy="55" r="5" fill="#e99d9b" opacity=".7"/><circle cx="80" cy="55" r="5" fill="#e99d9b" opacity=".7"/>
              </g>
            </svg>
            <span class="picture-label">{{ card.label }}</span>
            <span class="picture-category">{{ card.category }}</span>
          </button>
        </div>
        <div class="dice-controls">
          <p><span class="tiny-sparkle">✦</span> 选中 {{ lockedIds.length }} 个图案已锁定，再掷一次试试！</p>
          <button class="button-primary" @click="rollDice"><span>↻</span> 再掷一次</button>
        </div>
        <div class="prompt-strip"><span class="prompt-icon">✎</span><div><b>故事开头小帮手</b><p>{{ starterLines[starterIndex] }}</p></div><div class="prompt-actions"><button class="text-button" @click="randomStarter">换一句 ↗</button><button class="text-button" @click="startDraft">用这些灵感开写 ↗</button></div></div>
      </section>

      <section v-else class="workspace">
        <div class="section-heading">
          <div><span class="section-kicker">02 · OPEN A SURPRISE</span><h2>故事盲盒</h2><p>打开五个小盒子，让意想不到的事情发生！</p></div>
          <button class="button-soft" @click="rollAllBox">⟳ 全部重抽</button>
        </div>
        <div class="blindbox-grid">
          <article v-for="category in blindboxCategories" :key="category.id" class="blindbox-card" :class="'tone-' + category.color">
            <div class="blindbox-top"><span class="blindbox-symbol">{{ {person:'人',place:'地',event:'事',time:'时',mood:'心'}[category.id] }}</span><span>{{ category.name }}</span><span class="box-spark">✦</span></div>
            <div class="blindbox-window"><span class="window-star">✧</span><strong>{{ blindbox[category.id] }}</strong><span class="window-dot">✦</span></div>
            <button class="reroll-button" @click="rollBox(category.id)">↻ 换一个灵感</button>
          </article>
        </div>
        <div class="blindbox-footer"><div><b>把这些线索串起来吧！</b><p>不必按顺序，也可以给故事加入新角色。</p></div><button class="button-primary" @click="startDraft">开始编故事 <span>→</span></button></div>
      </section>

      <section v-if="!showSaved" id="story-editor" class="story-editor">
        <div class="editor-heading"><div><span class="section-kicker">YOUR LITTLE MASTERPIECE</span><h2>我的故事小书</h2><p>不用担心讲得对不对，大胆想象就好。</p></div><div class="editor-stamp">小小<br>作家</div></div>
        <div class="editor-paper">
          <label for="story-title">给故事起个名字</label>
          <input id="story-title" v-model="storyTitle" maxlength="50" placeholder="比如：小龙的神秘任务" />
          <label for="story-content">从这里开始讲……</label>
          <textarea id="story-content" v-model="storyText" rows="6" placeholder="很久很久以前，有一个……"></textarea>
          <div class="editor-actions"><span>✦ 每个奇思妙想都值得收藏</span><button class="button-primary" @click="saveStory">♡ 收藏故事</button></div>
        </div>
      </section>

      <section class="how-to-play">
        <div class="how-title"><span class="section-kicker">PLAY TOGETHER</span><h2>这样玩，最有趣</h2><p>不需要讲得完美，开心地想象就好。</p></div>
        <div class="how-grid">
          <article><span class="step-number">01</span><div class="how-icon how-icon-yellow">✦</div><h3>抽取灵感</h3><p>看看今天抽到了谁、在哪里、发生了什么。</p></article>
          <article><span class="step-number">02</span><div class="how-icon how-icon-pink">♡</div><h3>轮流讲故事</h3><p>爸爸讲一句，孩子接一句，故事会越来越奇妙。</p></article>
          <article><span class="step-number">03</span><div class="how-icon how-icon-blue">✧</div><h3>收藏你的故事</h3><p>把喜欢的故事保存下来，下次还能接着讲。</p></article>
        </div>
      </section>
    </main>
    <footer class="footer"><div class="footer-brand">✦ 故事星球</div><p>想象没有边界，每个孩子都是自己的故事家。</p><span>Made for little storytellers ♡</span></footer>
    <div v-if="notice" class="toast" role="status">{{ notice }}</div>
  </div>
</template>
