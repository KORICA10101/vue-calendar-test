<template>
  <div class="calendar-root">
    <!-- Шапка: листание месяца и переключатель языка -->
    <div class="calendar-header">
      <button class="nav-btn" @click="prevMonth" aria-label="previous month">‹</button>

      <div class="month-year">
        {{ monthName }} {{ currentYear }}
      </div>

      <div class="header-actions">
        <button class="nav-btn" @click="nextMonth" aria-label="next month">›</button>
        <!-- Переключатель языка -->
        <button class="lang-btn" @click="toggleLang" :title="lang === 'ru' ? 'Переключить на English' : 'Переключить на Русский'">
          {{ lang === 'ru' ? 'EN' : 'RU' }}
        </button>
      </div>
    </div>

    <!-- День недели -->
    <div class="weekdays">
      <div v-for="(wd, i) in weekDays" :key="i" class="weekday">{{ wd }}</div>
    </div>

    <!-- Сетка дней -->
    <div class="days-grid" role="grid" aria-label="calendar">
      <div
        v-for="(day, idx) in days"
        :key="idx"
        role="gridcell"
        tabindex="0"
        :class="[
          'day-cell',
          { 'day--other': day.otherMonth, 'day--selected': isSelected(day.date), 'day--today': isToday(day.date) }
        ]"
        @click="onDayClick(day)"
        @keydown.enter.prevent="onDayClick(day)"
      >
        {{ day.number }}
      </div>
    </div>

    <!-- Отображение выбранной даты в человекоподобном формате -->
    <div class="selected-block" v-if="selectedFormatted">
      <strong>Выбрана дата:</strong>
      <div class="selected-text">{{ selectedFormatted }}</div>
    </div>
  </div>
</template>

<script setup>
/* eslint-disable no-undef */
import { ref, computed, watch } from 'vue'

// ==== props и emits ====
const props = defineProps({
  modelValue: { type: String, default: null }, 
  langInit: { type: String, default: 'ru' }   // язык по умолчанию
})
const emit = defineEmits(['update:modelValue', 'select'])

// ==== локальное состояние ====
const lang = ref(props.langInit === 'en' ? 'en' : 'ru') // текущий язык
const today = new Date() 
const selectedDate = ref(props.modelValue ? parseDateString(props.modelValue) : new Date(today)) 
const currentYear = ref(selectedDate.value.getFullYear()) 
const currentMonth = ref(selectedDate.value.getMonth())   

const locales = {
  ru: {
    months: ['Январь','Февраль','Март','Апрель','Май','Июнь','Июль','Август','Сентябрь','Октябрь','Ноябрь','Декабрь'],
    weekdays: ['Пн','Вт','Ср','Чт','Пт','Сб','Вс'],
    formatDate: d => {
      const monthsGen = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
      return `${d.getDate()} ${monthsGen[d.getMonth()]} ${d.getFullYear()}`
    }
  },
  en: {
    months: ['January','February','March','April','May','June','July','August','September','October','November','December'],
    weekdays: ['Mo','Tu','We','Th','Fr','Sa','Su'],
    formatDate: d => d.toLocaleDateString('en-US', { day: 'numeric', month: 'long', year: 'numeric' })
  }
}

// ==== computed свойства ====
const monthName = computed(() => locales[lang.value].months[currentMonth.value])
const weekDays = computed(() => locales[lang.value].weekdays)

const days = computed(() => {
  const first = new Date(currentYear.value, currentMonth.value, 1)
  const last = new Date(currentYear.value, currentMonth.value + 1, 0)
  const startIdx = (first.getDay() + 6) % 7 // Пн = 0

  const out = []

  // хвост предыдущего месяца
  const prevMonthLastDate = new Date(currentYear.value, currentMonth.value, 0).getDate()
  for (let i = startIdx - 1; i >= 0; i--) {
    const d = new Date(currentYear.value, currentMonth.value - 1, prevMonthLastDate - i)
    out.push({ number: d.getDate(), date: d, otherMonth: true })
  }

  // дни текущего месяца
  for (let day = 1; day <= last.getDate(); day++) {
    const d = new Date(currentYear.value, currentMonth.value, day)
    out.push({ number: day, date: d, otherMonth: false })
  }

  // хвост следующего месяца
  while (out.length % 7 !== 0) {
    const nextIndex = out.length
    const d = new Date(currentYear.value, currentMonth.value + 1, nextIndex - last.getDate() - startIdx + 1)
    out.push({ number: d.getDate(), date: d, otherMonth: true })
  }

  return out
})

// ==== хелперы ====
function isSameDate(a, b) {
  return a.getFullYear() === b.getFullYear() && a.getMonth() === b.getMonth() && a.getDate() === b.getDate()
}
function isSelected(d) { return selectedDate.value && isSameDate(selectedDate.value, d) }
function isToday(d) { return isSameDate(today, d) }

// ==== навигация по месяцам ====
function prevMonth() {
  if (currentMonth.value === 0) { currentMonth.value = 11; currentYear.value-- } 
  else { currentMonth.value-- }
}
function nextMonth() {
  if (currentMonth.value === 11) { currentMonth.value = 0; currentYear.value++ } 
  else { currentMonth.value++ }
}

// ==== клик по дню ====
function onDayClick(day) {
  const d = day.date
  selectedDate.value = new Date(d.getFullYear(), d.getMonth(), d.getDate())
  if (d.getMonth() !== currentMonth.value || d.getFullYear() !== currentYear.value) {
    currentMonth.value = d.getMonth()
    currentYear.value = d.getFullYear()
  }
  emit('update:modelValue', toDateString(selectedDate.value))
  emit('select', new Date(selectedDate.value))
}

// ==== выбранная дата в формате человекоподобном ====
const selectedFormatted = computed(() => selectedDate.value ? locales[lang.value].formatDate(selectedDate.value) : '')

// ==== переключение языка ====
function toggleLang() {
  lang.value = lang.value === 'ru' ? 'en' : 'ru'
}

// ==== watch внешнего modelValue ====
watch(() => props.modelValue, (val) => {
  if (val) {
    const parsed = parseDateString(val)
    if (!isNaN(parsed)) {
      selectedDate.value = parsed
      currentYear.value = parsed.getFullYear()
      currentMonth.value = parsed.getMonth()
    }
  } else {
    selectedDate.value = new Date(today)
    currentYear.value = today.getFullYear()
    currentMonth.value = today.getMonth()
  }
})

// ==== утилиты ====
function toDateString(d) {
  const y = d.getFullYear()
  const m = String(d.getMonth() + 1).padStart(2, '0')
  const day = String(d.getDate()).padStart(2, '0')
  return `${y}-${m}-${day}`
}
function parseDateString(s) {
  if (!s) return new Date(NaN)
  const parts = String(s).split('-')
  if (parts.length !== 3) return new Date(NaN)
  const y = parseInt(parts[0], 10)
  const m = parseInt(parts[1], 10) - 1
  const d = parseInt(parts[2], 10)
  return new Date(y, m, d)
}
</script>


<style scoped>

.calendar-root {
  width: 320px;
  background: #ffffff;
  border: 1px solid #e6e6e6;
  padding: 12px;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  user-select: none;
}

/* Хедер: месяц и навигация */
.calendar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}
.month-year {
  font-weight: 600;
  font-size: 16px;
}
.header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Кнопки навигации */
.nav-btn {
  background: transparent;
  border: none;
  font-size: 20px;
  cursor: pointer;
  padding: 6px;
  line-height: 1;
}
.nav-btn:focus {
  outline: 2px solid rgba(0,0,0,0.08);
  border-radius: 6px;
}

/* Кнопка языка */
.lang-btn {
  background: #f3f4f6;
  border: 1px solid #e6e6e6;
  padding: 6px 8px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
}

/* Дни  */
.weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
  margin-bottom: 6px;
  text-align: center;
}
.weekday {
  font-weight: 600;
  color: #444;
  font-size: 13px;
}

/* Сетка дней */
.days-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
}
.day-cell {
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  cursor: pointer;
  padding: 4px;
  user-select: none;
  /* transition: background-color .12s ease; */
}
.day-cell:hover {
  background: #f5f7fb;
}


.day--other {
  color: #9aa4ad;
}
.day--selected {
  background: #1967d2;
  color: #fff;
  border-radius: 0px;
}
.day--today {
  /* box-shadow: inset 0 0 0 2px rgba(25,103,210,0.08); */
  border:1px solid blue;
  border-radius: 0px;
  
}

/* Блок выбранной даты */
.selected-block {
  margin-top: 12px;
  text-align: center;
  font-size: 14px;
}
.selected-text {
  margin-top: 6px;
  font-weight: 600;
}
</style>
