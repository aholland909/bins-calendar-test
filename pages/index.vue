<template>
  <div class="container">
    <div class="binform">
      <h1 class="subtitle">Bin test calendar</h1>
      <h1 class="subtitle">rg304tr</h1>
      <b-field class="mainsearch">
        <b-autocomplete
          class="searchbox"
          size="is-large"
          v-model="name"
          ref="autocomplete"
          :data="data"
          :loading="isFetching"
          :open-on-focus="true"
          :clear-on-select="false"
          placeholder="Enter postcode"
          @typing="getAsyncData"
          @select="selectAddress"
        >
          <template slot-scope="props">
            <div class="media">
              <div class="media-content search-result">{{props.option.SiteShortAddress}}</div>
            </div>
          </template>

          <template slot="empty">No results for {{name}}</template>
          <template slot="footer">
            <a @click="addressFindError">
              <span>Can't find address?</span>
            </a>
          </template>
        </b-autocomplete>
        <b-button
          @click.stop="$refs.autocomplete.focus()"
          size="is-large"
          type="is-primary"
          style="width:10%"
        >
          <b-icon icon="magnify" size="is-medium" class="material-icons"></b-icon>
        </b-button>
      </b-field>
      <div class="slide-container">
        <b-field>
          <!-- <div class="slide-controls">
            <b-switch v-model="byCalendar" @input="switchCheck(byCalendar, byDateRange)">By calendar</b-switch>
          </div>
          <div class="slide-controls">
            <b-switch
              v-model="byDateRange"
              @input="switchCheck(byCalendar, byDateRange)"
            >By date range</b-switch>
          </div>-->
          <div class="slide-controls">
            <b-button style="color:white" @click="gotoFullYear()">Get full year</b-button>
            <!-- <b-switch v-model="getYear" @input="yearOutput()">Get year data</b-switch> -->
          </div>
        </b-field>
      </div>

      <div v-if="byCalendar" style="display: inline-block;">
        <br />
        <b-datepicker inline v-model="date" :events="collectionDisplay" indicators="bars"></b-datepicker>
        <br />
      </div>

      <div v-if="byDateRange" style="display: inline-block;">
        <b-field label="Select a date">
          <b-datepicker
            ref="datepicker"
            placeholder
            :mobile-native="false"
            :events="collectionDisplay"
            :date-formatter="dateFormat"
            v-model="dates"
            @input="dateChange()"
            range
          ></b-datepicker>
        </b-field>
        <br />
      </div>
      <!-- data -->
      <!-- <div v-if="selected">{{selected}}</div>
      <br />
      <div v-if="collections.length">{{collections}}</div>-->

      <div v-if="collectionsError">
        <strong>No Kerbside collection for {{selected.SiteShortAddress}}</strong>
      </div>
      <!-- output year collection to test -->
      <div v-if="getYear">
        <div v-for="c in collections" :key="c.id">
          <p style="font-size: 0.8rem;font-weight: 300;">{{c}}</p>
        </div>
      </div>
      <div v-if="collections.length && !getYear">
        <div v-for="c in collections" :key="c.id">
          <div class="tile is-ancestor">
            <div class="tile is-vertical is-parent">
              <div class="tile is-child box">
                <p class="title is-2">{{c.Service}}</p>
                <!-- <p>{{c.Day}}</p> -->
                <p>{{formatDate(c.Date)}}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import debounce from "lodash/debounce";
import defer from "promise-defer";

export default {
  data() {
    return {
      name: "",
      postcode: "",
      data: [],
      results: [],
      selected: null,
      isFetching: false,
      isExpanded: false,
      addresses: [],
      collections: [],
      longcollections: [],
      collectionsError: false,
      byCalendar: false,
      byDateRange: false,
      getYear: false,
      previosSwitch: "",
      collectionDisplay: [],
      date: [],
      dates: [],
      // wp: process.env.WP_URL,
      // base_url: process.env.BASE_URL,
    };
  },
  methods: {
    switchCheck(cal, date) {
      if (cal == false && date == true) {
        this.previosSwitch = "byDateRange";
      }
      if (cal == true && date == false) {
        this.previosSwitch = "byCalendar";
      }
      if (cal == true && date == true) {
        if (this.previosSwitch == "byDateRange") {
          this.byDateRange = false;
          this.previosSwitch = true;
          this.previosSwitch = "byCalendar";
        } else if (this.previosSwitch == "byCalendar") {
          this.byDateRange = true;
          this.byCalendar = false;
          this.previosSwitch = "byDateRange";
        }
      }
    },
    getAsyncData: debounce(function (name) {
      if (!name.length) {
        this.data = [];
        return;
      }
      if (this.valid_postcode(name)) {
        this.isFetching = true;
        axios
          .get("https://api.reading.gov.uk/" + "rbc/getaddresses/" + name)
          .then(({ data }) => {
            this.data = [];
            if (data.Addresses != null) {
              data.Addresses.forEach((item) => this.data.push(item));

              //sort addresses
              this.data = this.data.sort(this.compareAddress);
            }
          })
          .catch((error) => {
            this.data = [];
            throw error;
          })
          .finally(() => {
            this.isFetching = false;
          });
      } else {
        this.data = [];
      }
    }, 500),

    valid_postcode(postcode) {
      postcode = postcode.replace(/\s/g, "");
      var regex = /^[A-Z]{1,2}[0-9]{1,2} ?[0-9][A-Z]{2}$/i;
      return regex.test(postcode);
    },

    compareAddress(a, b) {
      // check for numberhood
      a = a.SiteShortAddress.split(",")[0];
      b = b.SiteShortAddress.split(",")[0];
      const numA = !isNaN(a);
      const numB = !isNaN(b);
      if (numA && numB) {
        return Number(a) - Number(b);
      }
      if (numA) return -1;
      if (numB) return 1;
      // check for wordhood
      const wordA = /^[a-zA-Z]+$/.test(a);
      const wordB = /^[a-zA-Z]+$/.test(b);
      if (wordA && wordB) {
        return a.localeCompare(b);
      }
      if (wordA || wordB) return (wordA && -1) || 1;

      return 1; //or whatever logic to sort within non-alphanumeric values
    },

    selectAddress(selected) {
      this.isFetching = true;
      this.postcode = this.name;
      this.selected = selected;
      axios
        .get(
          "https://api.reading.gov.uk/" +
            "rbc/mycollections/" +
            this.selected.AccountSiteUprn
        )
        .then(({ data }) => {
          this.collections = [];
          this.collectionDisplay = [];
          if (data.Error == "No results returned") {
            this.collectionsError = true;
          } else {
            data.Collections.forEach((item) => this.collections.push(item));
            this.collectionsError = false;

            data.Collections.forEach((item) =>
              this.collectionDisplay.push({
                date: this.dateFormatter(item.Date),
                type: this.getBinColors(item.Service),
              })
            );
            //first date and end date of datepicker
            this.dates.push(new Date());
            this.dates.push(
              this.collectionDisplay[this.collectionDisplay.length - 1].date
            );
          }
        })
        .catch((error) => {
          this.data = [];
          throw error;
        })
        .finally(() => {
          this.isFetching = false;
        });
    },
    getCollectionByDate(uprn, startDate, endDate, collectionsArray) {
      var deferred = defer();
      var formattedstartDate =
        new Date(startDate).getFullYear() +
        "-" +
        (new Date(startDate).getMonth() + 1) +
        "-" +
        new Date(startDate).getDate();
      console.log("formatted data " + formattedstartDate);
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
          // handle error
          console.log(error);
        });

      return deferred.promise;
    },
    formatDate(date) {
      // var d = new Date(date);
      return date.split(" ")[0];
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
    addressFindError() {
      this.$buefy.dialog.alert({
        title: "Can't find address?",
        message: "Are you sure it's a Reading postcode?",
        confirmText: "Ok!",
      });
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
    gotoFullYear() {
      if (this.selected == null) {
        this.$buefy.dialog.alert({
          title: "Oops!",
          message: "Please enter a postcode and select an address!",
          confirmText: "Ok!",
        });
      } else {
        //might not need postcode!
        this.$router.push({
          path: "/pdf/" + this.selected.AccountSiteUprn + "/" + this.postcode,
        });
      }
    },
    yearOutput() {
      console.log("getting year collection data");
      var today = new Date();

      var todayplusyear = new Date(
        new Date().setFullYear(new Date().getFullYear() + 1)
      );

      this.collections = [];
      if (this.selected != null) {
        this.getCollectionByDate(
          this.selected.AccountSiteUprn,
          today,
          todayplusyear,
          this.collections
        ).then(() => {
          // update other cal
          this.collectionDisplay = [];
          this.collections.forEach((item) =>
            this.collectionDisplay.push({
              date: this.dateFormatter(item.Date),
              type: this.getBinColors(item.Service),
            })
          );
        });
      } else {
        this.$buefy.dialog.alert({
          title: "Oops!",
          message: "Please enter a postcode and select an address!",
          confirmText: "Ok!",
        });
      }
    },
    dateChange() {
      this.collections = [];
      if (this.selected != null) {
        this.getCollectionByDate(
          this.selected.AccountSiteUprn,
          this.dates[0],
          this.dates[1],
          this.collections
        ).then(() => {
          // update other cal
          this.collectionDisplay = [];
          this.collections.forEach((item) =>
            this.collectionDisplay.push({
              date: this.dateFormatter(item.Date),
              type: this.getBinColors(item.Service),
            })
          );
        });
      } else {
        this.$buefy.dialog.alert({
          title: "Oops!",
          message: "Please enter a postcode and select an address!",
          confirmText: "Ok!",
        });
      }
    },
    dateFormat(date) {
      if (date.length == 2)
        return `${date[0].getDate()}/${
          date[0].getMonth() + 1
        }/${date[0].getFullYear()} - ${date[1].getDate()}/${
          date[1].getMonth() + 1
        }/${date[1].getFullYear()}`;
    },
  },
};
</script>

<style>
.card {
  margin-top: 100px !important;
}
.tile {
  flex-basis: auto;
}
.tile.is-vertical > .tile.is-child:not(:last-child) {
  margin-bottom: 1rem !important;
}
.searchbox {
  display: inline-block;
  width: 100%;
}

.mainsearch {
  display: flex !important;
  justify-content: left !important;
}

.binform {
  padding-top: 2rem;
  margin-left: 5px;
  margin-right: 5px;
  width: 80%;
}
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: left;
  align-items: flex-start;
  text-align: left;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #000000;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #000000;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
.bin-image {
  float: left;
  padding-top: 20px;
}
.box {
  border-radius: 0 !important;
  width: 100%;
}
.title.is-2 {
  font-size: 1.5rem;
}
p {
  font-size: 1.5rem;
  font-weight: 300;
}
.input {
  border-radius: none !important;
}
.button {
  background-color: black !important;
  border-radius: 0%;
}

.input-shadow {
  color: black !important;
}
@media only screen and (max-width: 700px) {
  .binform {
    width: 100%;
  }
}
</style>
