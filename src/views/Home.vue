<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="tertiary">
        <ion-title class="ion-text-center" style="font-family: Pristina">Sound Writer</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" color="light" overflow-scroll="true">
      <div v-if="sentences.length" style="position:fixed; padding: 0.7rem; background-color: #f4f5f8; width: 100%;">
        <ion-icon :icon="trash" style="font-size: 1.5rem" class="ion-float-end"
                  color="danger" @click="sentences=[]"></ion-icon>
<!--        <ion-icon :icon="copyOutline" style="font-size: 1.5rem" class="ion-float-end"-->
<!--                  color="primary" @click="Copy()"></ion-icon>-->
      </div>
      <div v-if="sentences.length"
           style="font-family: 'Century Gothic'; background-color: #e3edff; margin-top: 3.5rem; border-radius: 6px; user-select: text"
           class="ion-padding ion-margin">
        <ion-text v-for="sentence in sentences" :key="sentence">{{ sentence + ' ' }}</ion-text>
      </div>
      <ion-fab vertical="bottom" horizontal="end" slot="fixed">
        <ion-fab-button color="light" @click.stop="toggle ? endSpeechRecognition() : startSpeechRecognition()"
                        icon>
          <ion-icon :icon="toggle? stopCircle : micCircle" color="danger" size="large"></ion-icon>
          <!--            <v-btn-->
          <!--                dark-->
          <!--                @click.stop="toggle ? endSpeechRecognition() : startSpeechRecognition()"-->
          <!--                icon-->
          <!--                :color="!toggle ? 'grey' : (speaking ? 'red' : 'red darken-3')"-->
          <!--                :class="{'animated infinite pulse': toggle}"-->
          <!--            >-->
          <!--              <v-icon>{{ toggle ? 'mic_off' : 'mic' }}</v-icon>-->
          <!--            </v-btn>-->
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonFab,
  IonFabButton,
  IonHeader,
  IonIcon,
  IonPage,
  IonTitle,
  IonToolbar,
  IonText,
  toastController
} from '@ionic/vue';
import {micCircle, stopCircle, trash, copyOutline} from 'ionicons/icons'
import {defineComponent} from 'vue';

declare global {
  interface Window {
    webkitSpeechRecognition: any;
  }
}

const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
const recognition = SpeechRecognition ? new SpeechRecognition() : false
export default defineComponent({
  name: 'Home',
  data() {
    return {
      copied: '',
      error: false,
      speaking: false,
      toggle: false,
      runtimeTranscription: '',
      sentences: [],
    }
  },
  methods: {
    async Copy() {
      this.copied = window.getSelection().anchorNode.textContent;
      if (this.copied != '') {
        document.execCommand('copy')
        const toast = await toastController.create({
          color: 'success',
          duration: 1000,
          message: 'Copied',
        });

        await toast.present();
      }
    },
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1)
    },
    endSpeechRecognition() {
      recognition.stop()
      this.toggle = false
      this.$emit('speechend', {
        sentences: this.sentences,
        text: this.sentences.join('. ')
      })
    },
    startSpeechRecognition() {
      if (!recognition) {
        this.error = 'Speech Recognition is not available on this browser. Please use Chrome or Firefox'
        return false
      }
      this.toggle = true
      // recognition.lang = this.lang
      recognition.interimResults = true
      console.log(recognition)

      // recognition.addEventListener('speechstart', event => {
      //   this.speaking = true
      // })
      //
      // recognition.addEventListener('speechend', event => {
      //   this.speaking = false
      // })

      recognition.addEventListener('result', event => {
        console.log("result", event)
        this.runtimeTranscription = Array.from(event.results).map(result => result[0]).map(result => result.transcript).join('')
      })

      recognition.addEventListener('end', () => {
        console.log("end")
        if (this.runtimeTranscription !== '') {
          this.sentences.push(this.capitalizeFirstLetter(this.runtimeTranscription))
          this.$emit('update:text', `${this.text}${this.sentences.slice(-1)[0]}. `)
        }
        this.runtimeTranscription = ''
        recognition.stop()
        if (this.toggle) {
          // keep it going.
          recognition.start()
        }
      })
      recognition.start()
    },
  },
  components: {
    IonPage,
    IonContent,
    IonHeader,
    IonTitle,
    IonToolbar,
    IonFab,
    IonFabButton,
    IonIcon,
    IonText
  },
  setup() {
    return {micCircle, stopCircle, trash, copyOutline};
  }
});
</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}

</style>