<template>
  <div class="center-container">
    <h1>Status von Herb-E</h1>
    <div>
      <table class="table table-dark table-striped">
        <tbody>
          <tr>
            <td>Startzeitpunkt</td>
            <td>{{ startZeitDatum }}</td>
          </tr>
          <tr v-if="endZeitStamp != 0">
            <td>Endzeitpunkt</td>
            <td>{{ endZeitStamp }}</td>
          </tr>
          <tr>
            <td>Laufzeit</td>
            <td>{{ laufzeit }}</td>
          </tr>
          <tr>
            <td>Distanz</td>
            <td>{{ distanz }} Meter</td>
          </tr>
          <tr>
            <td>Zustand</td>
            <td>{{ zustand }}</td>
          </tr>
          <tr>
            <td>Foto</td>
            <td>
              <div v-if="imageURL.length > 5">
                <img class="plantIMG" :src="imageURL" />
              </div>
              <div v-if="imageURL.length < 5">
                Es wurde noch keine Pflanze fotografiert
              </div>
            </td>
          </tr>
          <tr>
            <td>Erkannte Pflanzenart</td>
            <td>{{ erkenntePflanze }}</td>
          </tr>
          <tr v-if="commonName.length > 1">
            <td>Gemeiner Name</td>
            <td>{{ commonName }}</td>
          </tr>
          <tr>
            <td style="width: 35%">Position der gleichen Pflanze</td>
            <td>{{ positionDerGleichenPflanze }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <br />
    <h3>Fortschritt von Herb-E</h3>
    <h4 style="float: left">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="26"
        height="26"
        fill="currentColor"
        class="bi bi-caret-right-square-fill"
        viewBox="0 0 16 16"
        style="margin-top: -3px"
      >
        <path
          d="M0 2a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V2zm5.5 10a.5.5 0 0 0 .832.374l4.5-4a.5.5 0 0 0 0-.748l-4.5-4A.5.5 0 0 0 5.5 4v8z"
        />
      </svg>
      Start
    </h4>
    <h4 style="float: right">
      Ende
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="26"
        height="26"
        fill="currentColor"
        class="bi bi-flag-fill"
        viewBox="0 0 16 16"
      >
        <path
          d="M14.778.085A.5.5 0 0 1 15 .5V8a.5.5 0 0 1-.314.464L14.5 8l.186.464-.003.001-.006.003-.023.009a12.435 12.435 0 0 1-.397.15c-.264.095-.631.223-1.047.35-.816.252-1.879.523-2.71.523-.847 0-1.548-.28-2.158-.525l-.028-.01C7.68 8.71 7.14 8.5 6.5 8.5c-.7 0-1.638.23-2.437.477A19.626 19.626 0 0 0 3 9.342V15.5a.5.5 0 0 1-1 0V.5a.5.5 0 0 1 1 0v.282c.226-.079.496-.17.79-.26C4.606.272 5.67 0 6.5 0c.84 0 1.524.277 2.121.519l.043.018C9.286.788 9.828 1 10.5 1c.7 0 1.638-.23 2.437-.477a19.587 19.587 0 0 0 1.349-.476l.019-.007.004-.002h.001"
        />
      </svg>
    </h4>
    <div class="progress" style="height: 20px; clear: both">
      <div
        class="progress-bar"
        :class="{
          'progress-bar-striped progress-bar-animated bg-info': !finished,
          'bg-info': finished,
        }"
        role="progressbar"
        :style="{ width: distanz + '%' }"
        aria-valuenow="25"
        aria-valuemin="0"
        aria-valuemax="100"
      ></div>
      <img src="../assets/herb-e_mirror.png" width="50" class="bg-info" />
    </div>
  </div>
</template>

<script>
import { io } from "socket.io-client";
export default {
  name: "Panel",
  data() {
    return {
      socket: io("https://prenh21-dbrunner.enterpriselab.ch"),
      timer: undefined,
      isConnected: false,
      startZeitDatum: "",
      startZeitStamp: null,
      endZeitStamp: null,
      laufzeit: "",
      distanz: "",
      zustand: "",
      erkenntePflanze: "noch undefiniert",
      commonName: "",
      positionDerGleichenPflanze: "noch undefiniert",
      finished: false,
      imageURL: "",
    };
  },
  methods: {
    start() {
      this.updateLaufzeit(Date.now());
      this.timer = setInterval(() => {
        this.updateLaufzeit(Date.now());
      }, 1000);
    },
    stop() {
      clearInterval(this.timer);
      if (this.endZeitStamp > 0) {
        var date = new Date(this.endZeitStamp);
        this.endZeitStamp = date.toLocaleTimeString();
      }
    },
    updateLaufzeit(zeitStamp) {
      var laufzeitInMilli = Math.floor(
        (zeitStamp - this.startZeitStamp) / 1000
      );
      if (laufzeitInMilli < 0) {
        laufzeitInMilli = 0;
      }
      this.laufzeit =
        Math.floor(laufzeitInMilli / 60).toString() +
        " Minuten " +
        (laufzeitInMilli % 60).toString() +
        " Sekunden";
    },
  },
  mounted() {
    this.socket.on("runUpdated", (run) => {
      this.endZeitStamp = run.endTimeStamp;
      this.finished = run.isFinished;
      this.startZeitDatum = run.dateTimeStamp;
      this.startZeitStamp = run.startTimeStamp;
      this.distanz = run.distance;
      this.zustand = run.state;
      this.erkenntePflanze = run.plantType;
      this.commonName = run.commonName;
      this.positionDerGleichenPflanze = run.plantMatchPosition;
      this.imageURL = run.imageURL;
      if (this.endZeitStamp > 0 && this.finished == true) {
        this.updateLaufzeit(this.endZeitStamp);
        this.stop();
        this.timer = undefined
      } else if (this.startZeitStamp != null) {
        if (this.timer == undefined) {
          this.start();
        }
      } else if(this.startZeitStamp == null){
        this.startZeitDatum = "noch undefiniert"
        this.laufzeit = 0
      }
    }),
      this.socket.on("connect", () => {
        console.log("Successfully connected to Websocket!");
      });
  },
  created() {},
};
</script>
<style scoped>
img.plantIMG {
  width: 95%;
}
</style>
