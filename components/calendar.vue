<template>
  <section class="pdf-content">
    <section class="pdf-item">
      <div class="binform-header">
        <div>
          <img class="binform-header-logo" src="~assets/images/RBC.png" alt="RBC Home" />
        </div>
        <div class="binform-address">
          <h1 class="title is-4" style="margin:0px">Collections for:</h1>
          <h2 class="title is-6">{{this.address.replace("%", " ")}}</h2>
        </div>
      </div>
      <div class="binform-pdf-key">
        <div class="key-icons">Key:</div>
        <div class="key-icons">
          Food waste
          <b-icon icon="checkbox-blank" size="is-small" class="material-icons" type="is-twitter"></b-icon>
        </div>
        <div class="key-icons">
          Recycling
          <b-icon icon="checkbox-blank" size="is-small" class="material-icons" type="is-recycling"></b-icon>
        </div>
        <div class="key-icons">
          Rubbish
          <b-icon icon="checkbox-blank" size="is-small" class="material-icons" type="is-rubbish"></b-icon>
        </div>
        <div class="key-icons">
          Garden
          <b-icon icon="checkbox-blank" size="is-small" class="material-icons" type="is-garden"></b-icon>
        </div>
        <!-- <p>{{path}}</p> -->
        <!-- new Date().setFullYear(new Date().getFullYear() + 1) -->
      </div>
      <div class="bin-cal-container" ref="element">
        <div class="bin-cal" v-for="n in 12" :key="n">
          <b-datepicker
            :date-creator="() => { let date = new Date();
              return new Date(date.setMonth(new Date().getMonth() + n-1));}"
            inline
            size="is-size-7"
            :mobile-native="false"
            :events="collectionDisplay"
            indicators="bars"
            class="bin-cal-datepicker"
          >
            <template slot="header">
              <span class>{{formatTitle(n)}}</span>
            </template>
          </b-datepicker>
        </div>
      </div>
      <div class="binform-footer">
        <h1 class="title is-6" style="margin:0px">Dates subject to change</h1>
      </div>
      <!-- <div v-for="c in collections" :key="c.id">
        <p style="font-size: 0.8rem;font-weight: 300;">{{c}}</p>
      </div>-->
    </section>
  </section>
</template>

<script>
import axios from "axios";
import debounce from "lodash/debounce";
import defer from "promise-defer";

export default {
  props: {
    uprn: {
      type: String,
      required: false,
    },
    postcode: {
      type: String,
      required: false,
    },
    address: {
      type: String,
      required: false,
    },
  },
  data() {
    return {
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
      collections: [],
      collectionDisplay: [],
    };
  },

  mounted() {
    //start getcollections
    this.yearOutput();
  },

  methods: {
    formatTitle(m) {
      var cal = new Date();
      cal.setMonth(cal.getMonth() + m - 1);
      return this.monthNames[cal.getMonth()] + " " + cal.getFullYear();
    },
    yearOutput() {
      //maybe needs to the first day of month?
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
          this.$emit("domRendered");
          // this.loadingComponent.close();
        });
      } else {
        this.$buefy.dialog.alert({
          title: "Oops!",
          message: "Please enter a postcode and select an address!",
          confirmText: "Ok!",
        });
        //add redirect if needed
        // this.$router.push({
        //   path: "/",
        // });
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

<style>
.pdf-content {
  display: flex;
  width: 100%;
  height: auto;
  background: white;
}
.pdf-item {
  display: flex;
  flex-direction: column;
  background-color: white;
  /* width: 100%; */
  width: 210mm;

  height: auto;
}
.binform-footer {
  padding-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.binform-header-logo {
  height: 80px;
}
.binform-address {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.binform-header {
  display: flex;
  justify-content: space-between;
  margin-left: 20px;
  margin-right: 20px;
}
.key-icons {
  padding-right: 20px;
}
.binform-pdf-key {
  margin-left: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.bin-cal {
  display: flex;
  padding-top: 0.5rem;
}
.bin-cal-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-left: 20px;
  margin-right: 20px;
}
.binform-pdf {
  display: flex;
  flex-direction: column;
  background-color: white;
  height: unset;
  /* width: 210mm; */
  /* height: 100px; */
}
.bin-cal-datepicker{
  border-style: solid;
  border-color: #bebdbd;
  border-radius: 1px;
}
.dropdown-item,
.dropdown .dropdown-menu .has-link a {
  color: #4a4a4a;
  display: block;
  font-size: 0.875rem;
  line-height: 1.5;
  padding: 0.375rem 0.5rem;
  padding-top: 0rem;
  position: relative;
}
.datepicker .datepicker-header {
  padding-bottom: 0.2rem;
  margin-bottom: 0rem;
  border-bottom: 1px solid #dbdbdb;
}
.datepicker .datepicker-table .datepicker-cell {
  text-align: center;
  vertical-align: middle;
  display: table-cell;
  border-radius: 4px;
  padding: 0.3rem 0.5rem !important;
}
.datepicker .datepicker-table .datepicker-body.has-events .datepicker-cell {
  padding: 0.3rem 0.5rem 0.6rem;
}

/* position of event */
.datepicker .datepicker-table .datepicker-body.has-events .datepicker-cell.has-event .events {
    bottom: .225rem;
    display: flex;
    justify-content: center;
    left: 0;
    padding: 0 .35rem;
    position: absolute;
    width: 100%;
}

/* full height box */
.datepicker
  .datepicker-table
  .datepicker-body.has-events
  .datepicker-cell.has-event.bars
  .event {
  height: 1.75em;
  width: 100%;
}

.datepicker
  .datepicker-table
  .datepicker-body.has-events
  .datepicker-cell.has-event {
  color: white;
}

span {
  font-style: inherit;
  font-weight: inherit;
  z-index: 2;
  position: relative;
}
</style>