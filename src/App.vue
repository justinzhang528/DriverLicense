<template>
  <ion-app>
    <ion-router-outlet/>
    <ion-nav :root="homePage"></ion-nav>
    <SideMenu/>
  </ion-app>
</template>

<script setup lang="ts">
import {
  IonApp,
  IonRouterOutlet,
  IonNav,
  useBackButton,
} from '@ionic/vue';
import HomePage from "@/views/Page/HomePage.vue";
import {markRaw, onMounted, ref} from "vue";
import {useI18n} from "vue-i18n";
import useInternetConnection from "@/hooks/useInternetConnection";
import {showLoading} from "@/hooks/useUtils";
import scheduleNotification from "@/hooks/useLocalNotification";
import SideMenu from "@/views/SideMenu/SideMenu.vue";

const {isOnline} = useInternetConnection();
const {t,locale} = useI18n();
const homePage = markRaw(HomePage)

const checkInternetConnection = ()=> {
  if(!isOnline.value) {
    showLoading(t('noInternet'), ()=>{}, 5000);
  }
}

useBackButton(-1, () => {
  const nav = document.querySelector('ion-nav');
  nav?.pop();
});

onMounted(()=> {
  locale.value = localStorage.getItem("currentLanguage") || 'en';
  setInterval(checkInternetConnection, 5000);
  scheduleNotification(t('cycleLicense'),t('timeToStudy'));
});
</script>

<style>
  ion-thumbnail {
    --size: 40px;
  }
</style>