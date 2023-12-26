<template>
  <!-- <f7-chip smart-select :smart-select-params="{ openIn: 'sheet' }">
    <template #media>
      <img :src="$t('img')" />
    </template>
    <select @change="changeLanguage($event)" name="language" v-model="key">
      <option
        v-for="(lang, i) in langs"
        :key="`lang-${i}`"
        :value="lang"
        :selected="lang === selectedLang"
      >
        {{ $t(lang) }}
      </option>
    </select>
  </f7-chip> -->
  <f7-block>
      {{i18n.global.themeLocale}} theme
      <img v-if="theme === 'moon'" class="theme-select" src="../static/png/moon.png" width="25" height="25" @click="changeTheme('sun')" />
      <img v-else src="../static/png/sun.png" class="theme-select" width="25" height="25"  @click="changeTheme('moon')"/>
  </f7-block>
</template>

<script>
    import { onMounted, ref, computed } from 'vue';
    import { i18n,defaultTheme } from '../js/app';

export default {
  setup() {
    let selectedLang = i18n.global.locale;
    let theme = ref('sun');
    const langs = ['ja', 'es', 'en'];

    // Change language function
    const changeLanguage = (event) => {
      i18n.global.locale = event.target.value;
      selectedLang = event.target.value;
    };

    const changeTheme = (theme) => {
        alert(theme);
    theme = theme
    
    }
    onMounted(() => {
  alert(i18n.global.themeLocale, 'default')
    })
    return {
      changeLanguage,
      selectedLang,
      theme,
      changeTheme,
      langs,
    };
  },
};
</script>
<style scoped>
.theme-select {
cursor:pointer
}
    </style>