<template>
  <div class="container">
    <div class="binform">
      <div style="display:inline">
        <b-button>
          <nuxt-link to="/" style="color:white">Go home</nuxt-link>
        </b-button>
        <p>{{path}}</p>
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
// yarn add promise-defer
export default {
  data() {
    return {
      path: this.$route.params.pathMatch,
      uprn: "",
      postcode: "",
      collections: [],
      collectionDisplay: [],
    };
  },
  mounted() {
    var pathArray = this.path.split("/");
    this.uprn = pathArray[0];
    this.postcode = pathArray[1];
    this.yearOutput();
  },
  methods: {
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
</style>