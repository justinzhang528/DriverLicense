<template>
  <IonHeader>
    <IonToolbar>
      <IonButtons slot="start">
        <IonBackButton :text="$t('back')"></IonBackButton>
      </IonButtons>
      <IonButtons slot="end">
        <IonMenuButton></IonMenuButton>
      </IonButtons>
      <IonTitle class="center">
        <IonLabel>{{$t('rule')}}</IonLabel>
      </IonTitle>
    </IonToolbar>
  </IonHeader>
  <IonContent ref="contentRef" :scrollEvents="true" @ionScroll="onScroll">
<!--    <RecycleScroller class="scroller" :items="list" :item-size="300">-->
<!--      <template #default="{ item }">-->
<!--        <IonCard>-->
<!--          <IonIcon v-if="ruleBookmarkedItems.includes(item)" size="large" style="float: right; margin: 4px" :icon="bookmark" @click="onClickBookmarkIcon(item)"/>-->
<!--          <IonIcon v-if="!ruleBookmarkedItems.includes(item)" size="large" style="float: right; margin: 4px" :icon="bookmarkOutline" @click="onClickBookmarkIcon(item)"/>-->
<!--          <IonCardHeader>-->
<!--            <IonCardSubtitle class="center" style="padding-left: 40px">{{ item }}/{{ ruleCounts }}</IonCardSubtitle>-->
<!--          </IonCardHeader>-->
<!--          <IonCardContent>-->
<!--            <IonIcon class="iconBtn" size="large" :icon="playCircleOutline" @click="playRuleSound(item-1)"/>-->
<!--            <IonLabel style="color: black; font-weight: bold; padding-right: 5px">{{$t('question')}} {{$t(':')}}</IonLabel>-->
<!--            <IonLabel style="color: black;">{{ dataSource.rules[item-1].Q }}</IonLabel><br><br>-->
<!--            <IonIcon class="iconBtn" size="large" :icon="playCircleOutline" @click="playRuleSound(item-1)"/>-->
<!--            <IonLabel style="color: black; font-weight: bold; padding-right: 5px">{{$t('answer')}} {{$t(':')}} </IonLabel>-->
<!--            <IonLabel style="color: black;">{{ dataSource.rules[item-1].A }}</IonLabel>-->
<!--          </IonCardContent>-->
<!--        </IonCard>-->
<!--      </template>-->
<!--    </RecycleScroller>-->
      <IonCard v-for="i in ruleCounts" :key="i">
        <IonIcon v-if="ruleBookmarkedItems.includes(i)" size="large" style="float: right; margin: 4px" :icon="bookmark" @click="onClickBookmarkIcon(i)"/>
        <IonIcon v-if="!ruleBookmarkedItems.includes(i)" size="large" style="float: right; margin: 4px" :icon="bookmarkOutline" @click="onClickBookmarkIcon(i)"/>
        <IonCardHeader>
          <IonCardSubtitle class="center" style="padding-left: 40px">{{ i }}/{{ ruleCounts }}</IonCardSubtitle>
        </IonCardHeader>
        <IonCardContent>
          <IonIcon color="dark" v-if="!isPlayingRuleQuestionAudio[i-1]" size="large" :icon="playCircleOutline" @click="onClickPlayQuestionAudio(i-1)"/>
          <IonIcon color="dark" v-if="isPlayingRuleQuestionAudio[i-1]" size="large" :icon="pauseCircleOutline" @click="onClickPlayQuestionAudio(i-1)"/>
          <IonLabel color="dark" style="font-weight: bold; padding-right: 5px">{{$t('question')}} {{$t(':')}}</IonLabel>
          <IonLabel color="dark">{{ dataSource.rules[i-1].Q }}</IonLabel><br><br>
          <IonIcon color="dark" v-if="!isPlayingRuleAnswerAudio[i-1]" size="large" :icon="playCircleOutline" @click="onClickPlayAnswerAudio(i-1)"/>
          <IonIcon color="dark" v-if="isPlayingRuleAnswerAudio[i-1]" size="large" :icon="pauseCircleOutline" @click="onClickPlayAnswerAudio(i-1)"/>
          <IonLabel color="dark" style="font-weight: bold; padding-right: 5px">{{$t('answer')}} {{$t(':')}} </IonLabel>
          <IonLabel color="dark">{{ dataSource.rules[i-1].A }}</IonLabel>
        </IonCardContent>
      </IonCard>
  </IonContent>
</template>

<script setup lang="ts">
import {
  IonHeader,
  IonToolbar,
  IonButtons,
  IonBackButton,
  IonIcon,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardSubtitle,
  IonCardContent,
  IonTitle,
  IonLabel, IonMenuButton,
} from "@ionic/vue";
import {bookmark, bookmarkOutline, playCircleOutline, pauseCircleOutline} from "ionicons/icons";
import useData from '@/hooks/useData'
import {onMounted, onUnmounted, reactive, ref} from "vue";
import dataSource from '@/json/dataSource.json'
import {useI18n} from "vue-i18n";
import useAudio from "@/hooks/useAudio";
import {showToast} from "@/hooks/useUtils";

const {playRuleQuestionAudio, playRuleAnswerAudio, isPlayingRuleQuestionAudio, isPlayingRuleAnswerAudio, pauseAudio} = useAudio();
const {t} = useI18n();
const contentRef = ref();
const {addOrRemoveFromArray, ruleCounts, getBookmarkedItems} = useData()
const ruleBookmarkedItems = reactive(getBookmarkedItems('ruleBookmarkedItems'))

const onClickBookmarkIcon = (n: number) => {
  if (ruleBookmarkedItems.includes(n)) {
    showToast(t('removedFromBookmark'));
  } else {
    showToast(t('addedToBookmark'));
  }
  addOrRemoveFromArray(ruleBookmarkedItems,n);
}

const onScroll = (e: CustomEvent)=>{
  previousPosition.value = e.detail.currentY.toString();
}

const previousPosition = ref(parseInt(localStorage.getItem('ruleScrollPosition')) || 0);
const scrollToPreviousPosition = () => {
  contentRef.value.$el.scrollToPoint(0, previousPosition.value,150);
};

const onClickPlayQuestionAudio = (n: number) => {
  if(!isPlayingRuleQuestionAudio.value[n]){
    pauseAudio();
    const currentAudio = playRuleQuestionAudio(n);
    currentAudio.onended = () => {
      isPlayingRuleQuestionAudio.value[n] = false;
    }
  } else {
    pauseAudio();
  }
  for (let i = 0; i < ruleCounts; i++) {
    isPlayingRuleAnswerAudio.value[i] = false;
    if(i===n) continue;
    isPlayingRuleQuestionAudio.value[i] = false;
  }
  isPlayingRuleQuestionAudio.value[n] = !isPlayingRuleQuestionAudio.value[n];
}

const onClickPlayAnswerAudio = (n: number) => {
  if(!isPlayingRuleAnswerAudio.value[n]){
    pauseAudio();
    const currentAudio = playRuleAnswerAudio(n);
    currentAudio.onended = () => {
      isPlayingRuleAnswerAudio.value[n] = false;
    }
  } else {
    pauseAudio();
  }
  for (let i = 0; i < ruleCounts; i++) {
    isPlayingRuleQuestionAudio.value[i] = false;
    if(i===n) continue;
    isPlayingRuleAnswerAudio.value[i] = false;
  }
  isPlayingRuleAnswerAudio.value[n] = !isPlayingRuleAnswerAudio.value[n];
}

onMounted(()=>{
  scrollToPreviousPosition();
})

onUnmounted(()=>{
  localStorage.setItem('ruleBookmarkedItems', ruleBookmarkedItems.toString());
  localStorage.setItem('ruleScrollPosition', previousPosition.value.toString());
})

// const list = ref(Array.from({length: ruleCounts}, (_, i) => i + 1));

</script>

<style scoped>

</style>