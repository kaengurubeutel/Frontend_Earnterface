<script setup>
import {onMounted, ref} from 'vue';
import Card from './Card.vue';
defineProps({
})
// credits: https://github.com/TylerPottsDev/yt-vue-voice-recognition/blob/master/src/App.vue
//
let isRecording = ref(false);
let previousTexts = [];
let transscript = ref('');
let respondedText = ref('');
let receivedResponse = false;

const synth = new SpeechSynthesisUtterance();
const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition
const sr = new Recognition();

onMounted(() => {
  sr.continuous = true;
  sr.interimResults = true;
  sr.onstart = () => {
    console.log("started recording")
    isRecording.value = true;
  }
  sr.onend = () => {
    console.log('SR Stopped')
    isRecording.value = false;
  }

  sr.start()

  sr.onresult = (evt) => {
     for (let i = 0; i < evt.results.length; i++){
       const result = evt.results[i]

       if (result.isFinal) PassToRasa(result)
     }

    transscript.value = Array.from(evt.results)
        .map(result => result[0])
        .map(result => result.transcript)
        .join('')

  }



  })

    const PassToRasa = async function (input){

      let inputMessage = "";

      console.log(input[0].transcript)
      previousTexts[previousTexts.length] = [input[0].transcript, "false"];

      console.log(previousTexts)
      sr.stop();
      inputMessage = input[0].transcript;

      console.log(inputMessage)

      await fetch('http://localhost:5005/webhooks/rest/webhook', {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'charset':'UTF-8',
        },
        credentials: "same-origin",
        body: JSON.stringify({ "sender": "brian", "message": inputMessage }),
      })
          .then(response => response.json())
          .then((response) => {
            if (response) {
              let temp = response[0];
              let recipient_id = temp["recipient_id"];
              previousTexts[previousTexts.length]  = [temp["text"], "true"];


              synth.volume = 1; // 0 to 1
              synth.rate = 1; // 0.1 to 10
              synth.pitch = 1.5; // 0 to 2
              synth.text  = temp["text"];


              const voice =  {
                    "name": "Alex",
                    "lang": "en-US"
                  };

              synth.voiceURI = voice.name;
              synth.lang = voice.lang;
              speechSynthesis.speak(synth);

              sr.start()

            }
          })
    }
    const ToggleMic = () => {
      if(isRecording.value) {
        sr.stop();
      }else{
        sr.start();
      }
    }

</script>

<template>
  <h1>Rasa Terminal</h1>
  <button @click="ToggleMic">ToggleMic</button>
  <Card  :msg="text" bot="false"></Card>



  <div v-for =" text in previousTexts" :key="text.id" >
      <Card :msg="text[0]" :bot="text[1]"></Card>
  </div>


  <div id="rec" :class="{unvisible: !isRecording}"></div>
</template>

<style scoped>

#rec {
  width: 20px;
  height: 20px;
  position: fixed;
  left: 95vw;
  top: 90vh;
  background: #E02929;
  border-radius: 10px;

}

.unvisible{
  display: none;
}

</style>
