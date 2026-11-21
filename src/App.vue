<script setup>
import { ref } from 'vue'

const count = ref(5)
const length = ref(8)
const digits = ref(true)
const upper = ref(true)
const lower = ref(true)
const unique = ref(true)
const results = ref([])
const message = ref('')

const DIGITS = '0123456789'
const UPPER = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
const LOWER = 'abcdefghijklmnopqrstuvwxyz'

function alphabet() {
  return (
    (digits.value ? DIGITS : '') +
    (upper.value ? UPPER : '') +
    (lower.value ? LOWER : '')
  )
}

function randomIndex(max) {
  const buf = new Uint32Array(1)
  const limit = Math.floor(0x100000000 / max) * max
  let n = 0
  do {
    crypto.getRandomValues(buf)
    n = buf[0]
  } while (n >= limit)
  return n % max
}

function oneString(chars) {
  let out = ''
  for (let i = 0; i < length.value; i += 1) {
    out += chars[randomIndex(chars.length)]
  }
  return out
}

function notEnoughUnique(chars) {
  if (!unique.value) return false
  let space = 1
  for (let i = 0; i < length.value; i += 1) {
    space *= chars.length
    if (space >= count.value) return false
  }
  return true
}

function generate() {
  message.value = ''
  results.value = []
  const chars = alphabet()
  if (!chars) {
    message.value = '至少要勾一種字元。'
    return
  }
  if (!Number.isInteger(length.value) || length.value < 1 || length.value > 32) {
    message.value = '每個字串的長度要是 1 到 32 的整數。'
    return
  }
  if (!Number.isInteger(count.value) || count.value < 1 || count.value > 100) {
    message.value = '筆數要是 1 到 100 的整數。'
    return
  }
  if (notEnoughUnique(chars)) {
    message.value = '以現在的長度和字元集，湊不出這麼多不重複的字串。'
    return
  }

  const seen = new Set()
  const next = []
  let guard = 0
  while (next.length < count.value) {
    const item = oneString(chars)
    if (unique.value) {
      if (seen.has(item)) {
        guard += 1
        if (guard > 10000) {
          message.value = '不重複的字串不夠，請縮短筆數或加長字串。'
          return
        }
        continue
      }
      seen.add(item)
    }
    next.push(item)
  }
  results.value = next
}

async function copyAll() {
  if (!results.value.length) return
  try {
    await navigator.clipboard.writeText(results.value.join('\n'))
    message.value = '已複製全部。'
  } catch {
    message.value = '這個瀏覽器沒有複製成功，請直接選取結果。'
  }
}
</script>

<template>
  <main>
    <h1>隨機字串</h1>
    <p class="note">
      這一頁用瀏覽器的 crypto.getRandomValues 產生字元。它不是 random.org 的大氣噪聲。
    </p>

    <form @submit.prevent="generate">
      <label class="field">
        產生
        <input v-model.number="count" type="number" min="1" max="100" />
        筆
      </label>
      <label class="field">
        每個字串
        <input v-model.number="length" type="number" min="1" max="32" />
        個字
      </label>

      <div class="checks">
        <label class="field">
          <input v-model="digits" type="checkbox" />
          數字 0-9
        </label>
        <label class="field">
          <input v-model="upper" type="checkbox" />
          大寫 A-Z
        </label>
        <label class="field">
          <input v-model="lower" type="checkbox" />
          小寫 a-z
        </label>
      </div>

      <div class="checks">
        <label class="field">
          <input v-model="unique" type="radio" :value="true" />
          每筆都不重複
        </label>
        <label class="field">
          <input v-model="unique" type="radio" :value="false" />
          允許重複
        </label>
      </div>

      <button type="submit">產生</button>
    </form>

    <p class="message">{{ message }}</p>
    <button type="button" @click="copyAll">複製全部</button>

    <pre class="result"><template v-if="results.length">{{ results.join('\n') }}</template><template v-else>還沒有產生。</template></pre>
  </main>
</template>