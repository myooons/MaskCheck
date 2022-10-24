<template>
  <div class="template-box">
    <Head title="Camera" />
    <div class="content-box">
      <div id="title-container">마스크를 쓰셨나요?</div>

      <div id="webcam-container"></div>
      <div id="label-container"></div>
      <div id="result-container"></div>
    </div>
  </div>
</template>

<script>
import { Head } from "@inertiajs/inertia-vue";
import "@tensorflow/tfjs";
import * as tmImage from "@teachablemachine/image";

export default {
  components: {
    Head,
  },
  props: {
    products: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      model: null,
      webcam: null,
      labelContainer: null,
      maxPredictions: null,
      isIos: false,
      image: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    // Load the image model and setup the webcam
    async init() {
      const URL = "https://teachablemachine.withgoogle.com/models/27xOuCu9f/";

      const modelURL = URL + "model.json";
      const metadataURL = URL + "metadata.json";

      // load the model and metadata
      // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
      // or files from your local hard drive
      // Note: the pose library adds "tmImage" object to your window (window.tmImage)
      this.model = Object.freeze(await tmImage.load(modelURL, metadataURL));
      this.maxPredictions = this.model.getTotalClasses();

      // Convenience function to setup a webcam
      const flip = true; // whether to flip the webcam
      this.webcam = new tmImage.Webcam(250, 250, flip); // width, height, flip
      await this.webcam.setup(); // request access to the webcam
      await this.webcam.play();
      window.requestAnimationFrame(this.loop);

      // append elements to the DOM
      document
        .getElementById("webcam-container")
        .appendChild(this.webcam.canvas);
      this.labelContainer = document.getElementById("label-container");
      for (let i = 0; i < this.maxPredictions; i++) {
        // and class labels
        this.labelContainer.appendChild(document.createElement("div"));
      }
      this.resultContainer = document.getElementById("result-container");
      this.resultContainer.appendChild(document.createElement("div"));
    },
    async loop() {
      this.webcam.update(); // update the webcam frame
      await this.predict();
      window.requestAnimationFrame(this.loop);
    },
    async predict() {
      // run the webcam image through the image model
      // predict can take in an image, video or canvas html element
      console.log(this.webcam.canvas);
      const prediction = await this.model.predict(this.webcam.canvas);
      for (let i = 0; i < this.maxPredictions; i++) {
        const classPrediction =
          prediction[i].className + ": " + prediction[i].probability.toFixed(2);
        this.labelContainer.childNodes[i].innerHTML = classPrediction;

        if (prediction[0].probability > prediction[1].probability) {
          this.resultContainer.childNodes[0].innerHTML = "쓰셨군요!";
        } else {
          this.resultContainer.childNodes[0].innerHTML = "안쓰셨군요!";
        }
      }
    },
  },
};
</script>
<style>
@import url("https://fonts.googleapis.com/css2?family=Poor+Story&display=swap");

* {
  margin: 0;
  padding: 0;
}

.template-box {
  min-width: 375px;
  width: 100vw;
  height: 140vh;

  display: flex;
  justify-content: center;
  align-items: center;

  background-color: #42b883;
}

.content-box {
  width: 320px;
  height: 600px;
  border-radius: 1rem;

  display: flex;
  align-items: center;
  flex-direction: column;

  background-color: #ffffff;
}

#title-container {
  padding: 2.5rem;
  font-size: 1.8rem;
  font-family: "Poor Story", cursive;
}

#webcam-container {
  padding: 0.5rem;
  width: 250px;
  height: 250px;
  border: 1px solid #35495e;
}

#label-container {
  padding: 2rem;
  font-size: 1.5rem;
  font-family: "Poor Story", cursive;
}
#result-container {
  font-size: 3rem;
  font-family: "Poor Story", cursive;
}
</style>
