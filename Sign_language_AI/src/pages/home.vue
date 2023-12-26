<template>
  <f7-page name="home"  class="home">
    <!-- Top Navbar -->
    <!-- <f7-navbar>
      <f7-nav-left>
         <h1 class="title">{{ $t('title') }}</h1>
      </f7-nav-left>
      <f7-nav-right> -->
        <!-- <chip></chip> -->
           <!-- <f7-link href="/settings" class="link">
          <f7-icon f7="gear"></f7-icon>
        </f7-link> -->
        <!-- <img src="../static/png/moon.png" width="20" height="20" />
      </f7-nav-right>
    </f7-navbar> -->
    <!-- Page content-->
    <div v-if="!loaded" class="text-align-center loader">
      <div class="preloader"></div>
    </div>
    <f7-block class="main-container">
      <f7-row class="justify-content-center text-align-center camera-container">
        <f-card class="main-container">
          <div class="camera" v-show="!isMobile()">
            <video ref="videoRef" playsinline autoplay></video>
          </div>
          <div class="camera" v-show="isMobile()">
            <img ref="imageRef" src="static/sign.png" />
          </div>
        </f-card>
      </f7-row>
     <f7-block class="text-container">
           <f7-block  v-if="isResult" class="result-container">
              
            <h3 class="result">{{ result }}</h3>
        </f7-block>
    

        <f7-block class="start-btns">

             <f7-input v-if="isAllResult" type="text" placeholder="Enter text" class="result-container"> 
              <h5> {{ arr.join("") }} </h5>
            </f7-input>
            <f7-button v-if="!isResult" preloader :loading="isLoading" @click="showResult" fill class="start-btn"> Start Signing </f7-button>
            <f7-button v-else="isResult"  preloader :loading="isLoading" @click="stopSign" fill class="btn-red start-btn">Stop Signing</f7-button>
            <f7-input v-if="isAllResult" type="text" placeholder="Enter text" class="result-container"> 
                <h5> Translated Text(Akan):  {{  translatedText }} </h5>
              </f7-input>
              <f7-button v-if="isAllResult && arr.length > 0" @click="onTranslateClick" fill class="start-btn">Translate</f7-button>
        </f7-block>
        <!-- <f7-block  v-if="isAllResult" class="result-container">
            
        </f7-block> -->
    </f7-block>
  
     </f7-block>
  </f7-page>
</template>

<script>
import { onMounted, ref,watch, computed } from 'vue';
import * as tmImage from '@teachablemachine/image';
import { fetch as fetchPolyfill } from 'whatwg-fetch';
import { i18n } from '../js/app';
import axios from 'axios';
import { settings } from '../js/settings';
import chip from '../components/chip.vue';




const isAndroid = () => {
  const userAgent = navigator.userAgent || navigator.vendor || window.opera;
  if (/android/i.test(userAgent)) return true;
  return false;
};

const isIos = () => {
  const userAgent = navigator.userAgent || navigator.vendor || window.opera;
  if (/iPad|iPhone|iPod/i.test(userAgent)) return true;
  return false;
};

export default {
    
  components: {
    chip,
  },
  
  setup() {
    const videoRef = ref(null);
    const imageRef = ref(null);
    const usermediaLoaded = ref(false);
    const firstTimePredicted = ref(false);
    const user = ref('there is nothing');
    const result = ref(null);
     const isLoading = ref(false);
    const isResult = ref(false);
    const isAllResult = ref(false);
    const arr = ref([]);
    const translatedText = ref('');
    const width = 224;
    const height = 224;
    const predictionInterval = settings.getValue();
    const modalTime = 500;
    let modelLoaded = false;
    let model;
    let predictionTimer = null;
    let modelCheckLoop = null;
    let classes;
    let labels;

     const joinedString = computed(() => arr.value.join(''));

    // Watch the 'arr' variable if necessary
    watch(arr, () => {
      // You can perform additional actions when 'arr' changes if needed
    });

    // Access the value of joinedString somewhere else
    const accessedValue = joinedString.value;

    // Checks
    const isMobile = () => window.cordova && (isAndroid() || isIos());
 
    // Calculations
    const classify = (image) => {
      model.predict(image).then((prediction) => {
        let maxProbab = 0;
        let index = 0;
        for (let i = 0; i < classes; i++) {
          if (prediction[i].probability.toFixed(2) > maxProbab) {
            maxProbab = prediction[i].probability.toFixed(2);
            index = i;
          }
        }
        if (!firstTimePredicted.value) {
          firstTimePredicted.value = true;
        } else {
          user.value = prediction[index].className;
        }
      });
    };
   
    const calculate = () => {
      if (!user.value) {
        result.value = i18n.global.t('there is nothing');
        return;
      }
      if(isResult.value) {
            result.value = i18n.global.t(user.value);
            if(result.value != 'No sign detected.' && user.value != 'there is nothing' ) {
            arr.value.push(i18n.global.t(user.value));
        }
      }
        
       
    };

        const showResult = () => {
         if (isLoading.value) return;
            isLoading.value = true;
        setTimeout(() => {
            isLoading.value = false;
            isResult.value = true;
            }, 2000);
        isAllResult.value = false;
        calculate();
        arr.value = [];
      
    }
    const stopSign = () => {
        isResult.value = false;
       if (isLoading.value) return;
        isLoading.value = true;
      setTimeout(() => {
        isLoading.value = false;
      }, 1000);
      isAllResult.value = true;
        
    }

    // Browser
    const predictBrowser = () => {
      if (!model || !videoRef.value) {
        return;
      }
      predictionTimer = setInterval(() => {
        classify(videoRef.value);
        calculate();
      }, predictionInterval);
    };

    const startUserMedia = () => {
      const constraint = {
        audio: false,
        video: {
          facingMode: 'environment',
          width: {
            min: width,
            ideal: width,
            max: width,
            exact: width,
          },
          height: {
            min: height,
            ideal: height,
            max: height,
            exact: height,
          },
        },
      };
      const handleSuccess = (stream) => {
        usermediaLoaded.value = true;
        videoRef.value.srcObject = stream;
        predictBrowser();
      };
      const handleError = () => {
        usermediaLoaded.value = true;
      };
      navigator.mediaDevices
        .getUserMedia(constraint)
        .then(handleSuccess)
        .catch(handleError);
    };

    // Phone
    const readImageFile = (data) => {
      const protocol = 'file://';
      let filepath = '';
      if (isAndroid()) {
        filepath = protocol + data.output.images.fullsize.file;
      } else {
        filepath = data.output.images.fullsize.file;
      }
      window.resolveLocalFileSystemURL(
        filepath,
        async (fileEntry) => {
          fileEntry.file(
            (file) => {
              const reader = new FileReader();
              reader.onloadend = async () => {
                const blob = new Blob([new Uint8Array(reader.result)], {
                  type: 'image/png',
                });
                imageRef.value.src = window.URL.createObjectURL(blob);
              };
              reader.readAsArrayBuffer(file);
            },
            () => {},
          );
        },
        () => {},
      );
    };

    const startCanvasCamera = () => {
      const options = {
        canvas: {
          width,
          height,
        },
        capture: {
          width,
          height,
        },
        use: 'file',
        fps: 30,
        hasThumbnail: false,
        cameraFacing: 'front',
      };
      window.plugin.CanvasCamera.start(
        options,
        async () => {},
        (data) => {
          readImageFile(data);
        },
      );
    };

    const predictPhone = () => {
      if (!(model && window.plugin && window.plugin.CanvasCamera && imageRef)) {
        if (!(window.plugin && window.plugin.CanvasCamera)) {
          alert('window.plugin.CanvasCamera was not found');
        }
        return;
      }
      startCanvasCamera();
      predictionTimer = setInterval(() => {
        if (!imageRef || !imageRef.value || !imageRef.value.src) return;
        classify(imageRef.value);
        calculate();
      }, predictionInterval);
    };

    const start = async () => {
      if (isMobile()) {
        predictPhone();
      } else {
        predictBrowser();
      }
    };

    const stopLoop = () => {
      clearInterval(modelCheckLoop);
    };
 
    document.addEventListener(
      'deviceready',
      () => {
        modelCheckLoop = setInterval(() => {
          if (modelLoaded === true) {
            start();
            stopLoop();
          }
        }, modalTime);
      },
      false,
    );
    const translateText = async (textToTranslate) => {

      try {
        const response = await axios?.post("https://translation.googleapis.com/language/translate/v2?key=""", {
          q: textToTranslate, // text to translate here, this.textToTranslate
          target: 'ak', // Target language for Akan

        })
        // const response = await axios.post(`${API_URL}?key=${API_KEY}`, {
        //   q: "hello", // text to translate here, this.textToTranslate
        //   target: 'es', // Target language (e.g., Spanish)

        // })
       translatedText.value = response.data.data.translations[0].translatedText;
        
      } catch (error) {
        console.error('Error translating text:', error);
      }
    }

     const onTranslateClick = () => {
      translateText(joinedString.value);
    };
    // Setting up
    onMounted(async () => {
      if (isAndroid()) {
        window.fetch = fetchPolyfill;
      }
      const URL = 'static/model/';
      const modelURL = `${URL}model.json`;
      const metadataURL = `${URL}metadata.json`;

     
      model = await tmImage.load(modelURL, metadataURL);
      modelLoaded = true;
      classes = model.getTotalClasses();
      labels = model.getClassLabels();
      classify(imageRef.value);
      if (
        !isMobile()
        && navigator.mediaDevices
        && navigator.mediaDevices.getUserMedia
      ) {
        startUserMedia();
      }
    
    });
    
    
  // const transText = () => {
  //     console.log("tola")
  //   }
    const loaded = computed(() => {
      if (isMobile()) {
        return firstTimePredicted.value;
      }
      return usermediaLoaded.value && firstTimePredicted.value;
    });
    

    return {
      videoRef,
      imageRef,
      loaded,
      accessedValue,
      showResult,
      isLoading,
      translatedText,
      isResult,
      stopSign,
      user,
      onTranslateClick,
      translateText,
      start,
      isAllResult,
      arr,
      result,
      isMobile,
    };
  },
};
</script>

<style scoped>
.home {
    /* background-color: #4D4D4D; */
    /* margin: 0; */
}

.loader {
    margin-top: 20px;

}

.camera {
  height: 320px;
  width: 100vw;
  /* overflow: hidden; */
}

.camera video {
    width: 100%;
  /* border: 3px solid red; */
}

.main-container {
    padding-top: 0;
    margin-top: 0;
}

.text-container {
    /* border-top: 1px solid red; */
    /* padding-top: 5px; */
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    width: 100%;
    /* margin: 0; */
    background-color: #fff;
    /* box-shadow: 0 0 20px hsl(0deg 0% 0% / 0.1), 0 0 5px hsl(0deg 0% 0% / 0); */
   padding-bottom: 80px;
}

.start-btns {
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 4rem;
    padding-top: 5px;
}
.textbox{
    padding: 2rem;
    font-size: 18px;
}
.start-btn {
    padding: 1.5rem 2rem;
}

.btn-red {
    background: #F31559;
}

.result {
    padding-top: 10px;
    font-size: 15px;
}

.result-container {
  text-align: center;
}

.chip {
  width: 100px;
}
.link {
  color: #000000;
}

.title {
    width: 100%;
  text-align: center;
  padding: 10px 20px;
}
</style>
