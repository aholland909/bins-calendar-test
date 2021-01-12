<template>
  <div v-if="!isLoading" class="cal-container">
    <b-button class="bin-cal-print-button" @click="pdfgencss()">
      <div class="button-container">
        <b-icon icon="printer" custom-size="mdi-36px" class="material-icons"></b-icon>
        <span style="padding-left:1rem">Download PDF</span>
      </div>
    </b-button>

    <Calendar
      :uprn="this.uprn"
      :postcode="this.postcode"
      :address="this.address"
      @domRendered="domRendered()"
    />
  </div>
</template>

<script>
import Calendar from "@/components/Calendar";
import axios from "axios";
import debounce from "lodash/debounce";
import defer from "promise-defer";
// import * as jsPDF from "jspdf";

// yarn add promise-defer
export default {
  data() {
    return {
      loadingComponent: null,
      isLoading: true,
      isFullPage: true,
      monthNames: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ],
      path: this.$route.params.pathMatch,
      uprn: "",
      postcode: "",
      address: "",
      collections: [],
      collectionDisplay: [],
    };
  },
  components: {
    Calendar,
  },
  mounted() {
    this.loadingComponent = this.$buefy.loading.open(); //disable for testing
    var pathArray = this.path.split("/");
    this.uprn = pathArray[0];
    this.postcode = pathArray[1];
    this.address = pathArray[2];
    // this.yearOutput();
    this.isLoading = false;
  },

  methods: {
    pdfgencss() {
      if (process.browser) {
        window.print();
      }
    },
    domRendered() {
      console.log("Cal Has Rendered");
      this.isLoading = false; //disable for testing
      this.loadingComponent.close(); //disable for testing

      // setTimeout(() => {
      //       if (process.browser) {
      //         window.print();
      //       }
      //     }, 2000);
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
          this.collections.forEach((item) => {
            // console.log(item);
            this.collectionDisplay.push({
              date: this.dateFormatter(item.Date),
              type: this.getBinColors(item.Service),
            });
          });
          this.isLoading = false;
          this.loadingComponent.close();
          setTimeout(() => {
            if (process.browser) {
              window.print();
            }
          }, 2000);
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
              //add one day to new start date
              var oldStartDate = this.dateFormatter(
                collectionsArray[collectionsArray.length - 1].Date
              );
              startDate = oldStartDate.setDate(oldStartDate.getDate() + 1);

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
          console.log(error);
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
    getBinColors(serviceString) {
      if (serviceString == "Recycling Collection Service") {
        return "is-recycling";
      }
      if (serviceString == "Domestic Waste Collection Service") {
        return "is-rubbish";
      }
      if (serviceString == "Garden Waste Collection Service") {
        return "is-garden";
      }
      if (serviceString == "Food Waste Collection Service") {
        return "is-twitter";
      }
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

<style lang="scss">
.button-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.5rem;
}
.bin-cal-print-button{
  color:black;
  margin: 1.4rem;
} 
.cal-container {
  width: 220mm;
}

* {
  -webkit-print-color-adjust: exact !important; /*Chrome, Safari */
  color-adjust: exact !important; /*Firefox*/
}
@page {
  size: auto; /* auto is the initial value */
  width: 100%;
  height: 100%;
  /* this affects the margin in the printer settings */
  margin-top: 0mm;
  margin-bottom: 0mm;
  margin-right: 0mm;
  margin-left: 0mm;
}
@media print {
  .bin-cal-print-button {
    display: none;
  }
  .cal-container {
    width: 220mm !important;
  }
}
@media (max-width: 768px) {
  .cal-container{
    width: 100%;
  }
}
</style>