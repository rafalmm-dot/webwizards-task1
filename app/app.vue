<template>
  <main class="page">
    <section class="user">
      <p v-if="loading" class="user__message">
        Ładowanie danych...
      </p>

      <p v-else-if="error" class="user__message user__message--error">
        Nie udało się pobrać danych.
      </p>

      <div v-else class="user__box">
        <div class="user__header">
          <img
            class="user__photo"
            :src="imageSrc"
            :alt="user.name + ' ' + user.surname"
          >

          <div class="user__data">
            <h1 class="user__name">
              {{ user.name }} {{ user.surname }}
            </h1>

            <p class="user__text">
              <strong>Email:</strong>
              <a :href="'mailto:' + user.email" class="user__link">
                {{ user.email }}
              </a>
            </p>

            <p class="user__text">
              <strong>Numer telefonu:</strong>
              <span>{{ phoneToShow }}</span>

            <button
  class="user__small-btn"
  type="button"
  @click="showPhone = !showPhone"
>
  {{ showPhone ? 'Ukryj numer' : 'Pokaż numer' }}
</button>
            </p>
          </div>

          <button
            class="user__download"
            type="button"
            @click="downloadJson"
          >
            Pobierz dane użytkownika
          </button>
        </div>

        <div class="user__description">
          <div class="user__about">
            <p
              v-for="(paragraph, index) in visibleAbout"
              :key="index"
              class="user__paragraph"
            >
              {{ paragraph }}
            </p>
          </div>

<button
  class="user__more"
  type="button"
  @click="showAllAbout = !showAllAbout"
>
  {{ showAllAbout ? 'Zwiń opis' : 'Rozwiń opis' }}
</button>
        </div>
      </div>
    </section>
  </main> 
</template>

<script setup>
const apiUrl = 'https://webwizards.home.pl/jacek/frontend-task/api/user/'

const user = ref(null)
const apiData = ref(null)
const loading = ref(true)
const error = ref(false)

const showPhone = ref(false)
const showAllAbout = ref(false)

onMounted(async () => {
  try {
    const response = await fetch(apiUrl)
    const result = await response.json()

    apiData.value = result
    user.value = result.data
  } catch (err) {
    error.value = true
  } finally {
    loading.value = false
  }
})

const imageSrc = computed(() => {
  if (!user.value) {
    return ''
  }

  return user.value.image.baseUrl +
    user.value.image.filename +
    '.' +
    user.value.image.extension
})

const phoneToShow = computed(() => {
  if (!user.value) {
    return ''
  }

  const phone = String(user.value.phone)

  if (showPhone.value) {
    return phone
  }

  const lastNumbers = phone.slice(-3)
  const hiddenNumbers = 'X'.repeat(phone.length - 3)

  return hiddenNumbers + lastNumbers
})

const aboutParagraphs = computed(() => {
  if (!user.value) {
    return []
  }

  return user.value.about
    .split(/\n\s*\n/)
    .map(paragraph => paragraph.trim())
    .filter(paragraph => paragraph.length > 0)
})

const visibleAbout = computed(() => {
  if (showAllAbout.value) {
    return aboutParagraphs.value
  }

  return aboutParagraphs.value.slice(0, 1)
})

function downloadJson() {
  const fileContent = JSON.stringify(apiData.value, null, 2)
  const file = new Blob([fileContent], { type: 'application/json' })
  const fileUrl = URL.createObjectURL(file)

  const link = document.createElement('a')
  link.href = fileUrl
  link.download = 'user-data.json'
  link.click()

  URL.revokeObjectURL(fileUrl)
}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Roboto, Arial, sans-serif;
  background: #ffffff;
}

button,
a {
  font: inherit;
}

.page {
  min-height: 100vh;
  padding: 40px 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.user {
  width: 100%;
  max-width: 1344px;
  padding: 90px 70px;
  border-radius: 28px;
  background: #e7e9e9;

  &__message {
    margin: 0;
    color: #222222;
    font-size: 18px;
    text-align: center;

    &--error {
      color: #b00020;
    }
  }

  &__box {
    max-width: 980px;
    margin: 0 auto;
  }

  &__header {
    position: relative;
    min-height: 170px;
    padding: 32px 28px 28px 205px;
    display: flex;
    align-items: center;
    background: #243f78;
    border-radius: 10px;
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
  }

  &__photo {
    position: absolute;
    left: 28px;
    top: -22px;
    width: 155px;
    height: 155px;
    padding: 8px;
    object-fit: cover;
    border: 3px solid #d6f6ff;
    border-radius: 8px;
    background: #38bdf8;
  }

  &__data {
    color: #ffffff;
  }

  &__name {
    margin: 0 0 18px;
    font-size: 28px;
    line-height: 1.1;
  }

  &__text {
    margin: 0 0 10px;
    font-size: 14px;
  }

  &__link {
    color: #ffffff;
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }

  &__small-btn {
    margin-left: 6px;
    padding: 4px 7px;
    border: 0;
    border-radius: 3px;
    color: #ffffff;
    background: #38bdf8;
    font-size: 11px;
    font-weight: 700;
    cursor: pointer;
  }

  &__download {
    position: absolute;
    top: 24px;
    right: 24px;
    padding: 8px 12px;
    border: 0;
    border-radius: 4px;
    color: #ffffff;
    background: #38bdf8;
    font-size: 12px;
    font-weight: 700;
    cursor: pointer;
  }

  &__description {
    width: 90%;
    margin: -5px auto 0;
    padding: 45px 30px 25px;
    background: #4ab4e7;
    border-radius: 0 0 8px 8px;
    box-shadow: 0 12px 22px rgba(0, 0, 0, 0.18);
  }

  &__about {
    padding: 14px 16px;
    border: 1px solid #238bc2;
  }

  &__paragraph {
    margin: 0 0 14px;
    color: #0b2940;
    font-size: 16px;
    line-height: 1.5;
    font-weight: 600;

    &:last-child {
      margin-bottom: 0;
    }
  }

  &__more {
    margin-top: 12px;
    padding: 0;
    border: 0;
    color: #ffffff;
    background: transparent;
    font-size: 13px;
    font-weight: 700;
    cursor: pointer;

    &:hover {
      text-decoration: underline;
    }
  }
}

@media (max-width: 900px) {
  .user {
    padding: 60px 24px;

    &__header {
      padding: 150px 24px 28px;
      display: block;
      text-align: center;
    }

    &__photo {
      left: 50%;
      transform: translateX(-50%);
    }

    &__download {
      position: static;
      width: 100%;
      margin-top: 20px;
    }

    &__description {
      width: 95%;
    }
  }
}

@media (max-width: 560px) {
  .page {
    padding: 16px;
  }

  .user {
    padding: 45px 14px;
    border-radius: 20px;

    &__header {
      padding: 140px 16px 24px;
    }

    &__photo {
      width: 145px;
      height: 145px;
    }

    &__name {
      font-size: 24px;
    }

    &__description {
      width: 100%;
      padding: 30px 14px 20px;
    }
  }
}
</style>
