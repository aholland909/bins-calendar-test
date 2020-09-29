<template>
  <div class="container">
    <div class="binform" id="pdf">
      <div style="display:inline">
        <b-button>
          <nuxt-link to="/" style="color:white">Go home</nuxt-link>
        </b-button>
        <b-button style="color:white" @click="pdfgencss()">Download PDF</b-button>
        <!-- <p>{{path}}</p> -->
      </div>

      <div v-for="c in collections" :key="c.id">
        <p style="font-size: 0.8rem;font-weight: 300;">{{c}}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import debounce from "lodash/debounce";
import defer from "promise-defer";
import * as jsPDF from "jspdf";

// yarn add promise-defer
export default {
  data() {
    return {
      path: this.$route.params.pathMatch,
      uprn: "",
      postcode: "",
      collections: [],
      collectionDisplay: [],
      "a4-landscape": {
        orientation: "landscape",
        unit: "cm",
        format: "a4",
        maxWidth: 23,
        maxHeight: 18,
        left: 1.5,
        top: 3,
        space: 0.5,
        lineHeight: 0.5,
      },
    };
  },
  mounted() {
    var pathArray = this.path.split("/");
    this.uprn = pathArray[0];
    this.postcode = pathArray[1];
    this.yearOutput();
  },
  methods: {
    pdfgencss() {
      console.log("test pdf output with css");
      if (process.browser) {
        window.print();

        // const filename = this.uprn + ".pdf";
        // // const jsPDF = require("jspdf");
        // // let doc = new jsPDF("p", "cm", "a4");
        // const html2canvas = require("html2canvas");
        // var canvasElement = document.createElement("canvas");
        // html2canvas(document.querySelector("#pdf")).then((canvas) => {
        //   let pdf = new jsPDF("p", "mm", "a4");
        //   pdf.addImage(canvas.toDataURL("image/png"), "PNG", 0, 0, 211, 298);
        //   pdf.save(filename);
        // });
      }
    },
    yearOutput() {
      var today = new Date();

      var todayplusyear = new Date(
        new Date().setFullYear(new Date().getFullYear() + 1)
      );

      this.collections = [];
      if (this.uprn != null && this.uprn.length == 9) {
        this.getCollectionByDate(
          this.uprn,
          today,
          todayplusyear,
          this.collections
        ).then(() => {
          // update other cal
          this.collectionDisplay = [];
          this.collections.forEach((item) =>
            this.collectionDisplay.push({
              date: this.dateFormatter(item.Date),
              //   type: this.getBinColors(item.Service),
            })
          );
        });
      } else {
        this.$buefy.dialog.alert({
          title: "Oops!",
          message: "Please enter a postcode and select an address!",
          confirmText: "Ok!",
        });
        this.$router.push({
          path: "/",
        });
      }
    },
    getCollectionByDate(uprn, startDate, endDate, collectionsArray) {
      var deferred = defer();
      var formattedstartDate =
        new Date(startDate).getFullYear() +
        "-" +
        (new Date(startDate).getMonth() + 1) +
        "-" +
        new Date(startDate).getDate();
      //   console.log("formatted data " + formattedstartDate);
      axios
        .get(
          "https://api.reading.gov.uk/" +
            "rbc/mycollections/" +
            uprn +
            "/" +
            formattedstartDate
        )
        .then((response) => {
          // handle success
          if (response.data.Collections) {
            if (new Date(startDate) < new Date(endDate)) {
              response.data.Collections.forEach((item) =>
                collectionsArray.push(item)
              );
              startDate = this.dateFormatter(
                collectionsArray[collectionsArray.length - 1].Date
              );
              this.getCollectionByDate(
                uprn,
                startDate,
                endDate,
                collectionsArray
              ).then(function () {
                deferred.resolve();
              });
            } else {
              deferred.resolve();
            }
          }
        })
        .catch((error) => {
          // handle 404 error
          //   console.log(error);
          this.$buefy.dialog.alert({
            title: "Oops!",
            message: "Please enter a postcode and select an address!",
            confirmText: "Ok!",
          });
          this.$router.push({
            path: "/",
          });
        });

      return deferred.promise;
    },
    dateFormatter(dateString) {
      var reggie = /(\d{2})\/(\d{2})\/(\d{4}) (\d{2}):(\d{2}):(\d{2})/;
      var dateArray = reggie.exec(dateString);
      // console.log(dateArray)
      var dateObject = new Date(
        +dateArray[3],
        +dateArray[2] - 1, // Careful, month starts at 0!
        +dateArray[1],
        +dateArray[4],
        +dateArray[5],
        +dateArray[6]
      );
      return dateObject;
    },
  },
};
</script>

<style>
* {
  -webkit-print-color-adjust: exact !important; /*Chrome, Safari */
  color-adjust: exact !important; /*Firefox*/
}
@page {
  size: auto; /* auto is the initial value */
  margin-top: 0mm; /* this affects the margin in the printer settings */
  margin-bottom: 0mm; /* this affects the margin in the printer settings */
}
</style>