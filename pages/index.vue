<template>
  <div class="container" id="container">
    <div class="binform" id="binform">
      <h1 class="subtitle">Bin test calendar pdf test</h1>
      <b-field class="mainsearch" type="is-black">
        <b-autocomplete
          class="searchbox"
          size="is-large"
          v-model="name"
          ref="autocomplete"
          :data="data"
          :loading="isFetching"
          :open-on-focus="true"
          :clear-on-select="false"
          placeholder="e.g. RG1 2LU"
          @typing="getAsyncData"
          @select="selectAddress"
        >
          <template slot-scope="props">
            <div class="media">
              <div class="media-content search-result">
                <p>{{props.option.SiteShortAddress}}</p>
              </div>
            </div>
          </template>

          <template slot="empty">No results for {{name}}</template>
          <template slot="footer">
            <p>
              <a @click="addressFindError" style="color:black">
                <span>Your address not found?</span>
              </a>
            </p>
          </template>
        </b-autocomplete>
        <b-button
          @click.stop="$refs.autocomplete.focus()"
          size="is-large"
          type="is-primary"
          style="width:5%"
        >
          <b-icon icon="magnify" size="is-medium" class="material-icons"></b-icon>
        </b-button>
      </b-field>
      <div v-if="collections.length" class="slide-container" style="padding-bottom: 0.75rem;">
        <b-field>
          <div class="slide-controls">
            <b-button style="color:white" @click="gotoFullYear()">Get full year</b-button>
            <!-- <b-switch v-model="getYear" @input="yearOutput()">Get year data</b-switch> -->
          </div>
        </b-field>
      </div>

      <div v-if="collectionsError" class="tile is-child box">
        <p>No kerbside collection for {{selected.SiteShortAddress}}.</p>
        <p>If this property is a flat there will be an alternative collection arrangement.</p>
      </div>
      <!-- output year collection to test -->
      <div class="tile is-ancestor">
        <div v-if="collections.length" class="tile is-vertical is-parent">
          <div class="tile is-child box">
            <p class="title is-2">Address: {{selected.SiteShortAddress}}</p>
          </div>
          <div v-for="c in collections" :key="c.id" class="tile is-child box">
            <p class="title is-2">{{changeServiceName(c.Service)}}</p>
            <!-- <p>{{c.Day}}</p> -->
            <p>{{getDay(c.Date)}} - {{formatDate(c.Date)}}</p>
            <div class="bin-image" id="icon">
              <img class="image is-64x64" :src="getImage(c.Service)" fill="white" />
            </div>

            <b-collapse class="card" animation="slide" aria-id="contentIdForA11y3" :open="false">
              <div
                slot="trigger"
                slot-scope="props"
                class="card-header"
                role="button"
                aria-controls="contentIdForA11y3"
              >
                <p class="card-header-title" style="font-weight:300">What can I put in this bin?</p>
                <a class="card-header-icon">
                  <b-icon type="is-black" :icon="props.open ? 'menu-down' : 'menu-up'"></b-icon>
                </a>
              </div>
              <div class="card-content">
                <div class="content" v-for="i in binContent(c.Service)" :key="i.id">
                  <p>{{i}}</p>
                </div>
              </div>
            </b-collapse>
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
import jsPDF from "jspdf";

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
      rbcapi: process.env.RBCAPI_URL,
      // wp: process.env.WP_URL,
      // base_url: process.env.BASE_URL,
    };
  },
  mounted(){
    console.log(process.env)
    console.log(this.rbcapi)
  },
  methods: {
    pdfgencss() {
      console.log("test pdf output with css");
      if (process.browser) {
        window.print();
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
        var calendaraddress = this.selected.SiteShortAddress.replace(/,/g, "");
        this.$router.push({
          path:
            "/pdf/" +
            this.selected.AccountSiteUprn +
            "/" +
            this.postcode +
            "/" +
            calendaraddress,
        });
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
          .get(this.rbcapi + "rbc/getaddresses/" + name)
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
      this.collections = [];
      this.isFetching = true;
      this.selected = selected;
      axios
        .get(this.rbcapi + "rbc/mycollections/" + this.selected.AccountSiteUprn)
        .then(({ data }) => {
          this.collections = [];
          if (data.Error == "No results returned") {
            this.collectionsError = true;
          } else {
            data.Collections.forEach((item) => this.collections.push(item));
            this.collectionsError = false;
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
    getDay(date) {
      var days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      var d = this.dateFormatter(date);
      return days[d.getDay()];
    },
    addressFindError() {
      this.$buefy.dialog.alert({
        title: "Shucks! We may not be your council :-(",
        message:
          'Check who your local authority is <a href="https://www.gov.uk/find-local-council">here</a>',
        confirmText: "Close",
      });
    },
    changeServiceName(serviceName) {
      if (serviceName == "Recycling Collection Service") {
        return "Recycling (red bin)";
      }
      if (serviceName == "Domestic Waste Collection Service") {
        return "Rubbish (grey bin)";
      }
      if (serviceName == "Garden Waste Collection Service") {
        return "Garden (green bin)";
      }
      if (serviceName == "Food Waste Collection Service") {
        return "Food";
      }
    },
    getImage(serviceName) {
      if (serviceName == "Recycling Collection Service") {
        return require("~/assets/icons/" + "redWaste.svg");
      }
      if (serviceName == "Domestic Waste Collection Service") {
        return require("~/assets/icons/" + "greyWaste.svg");
      }
      if (serviceName == "Garden Waste Collection Service") {
        return require("~/assets/icons/" + "greenWaste.svg");
      }
      if (serviceName == "Food Waste Collection Service") {
        return require("~/assets/icons/" + "foodWaste.svg");
      }
    },
    binContent(serviceName) {
      if (serviceName == "Recycling Collection Service") {
        var array = [
          "✓ Plastic bottles (e.g. drinks, milk, toiletries, detergent)",
          "✓ Plastic pots (e.g. yoghurt, cream, snack, soup)",
          "✓ Plastic trays (e.g. fruit punnets, meat/cake trays)",
          "✓ Plastic tubs (e.g. ice cream, margarine, sweets tubs)",
          "✓ Paper and card",
          "✓ Cartons (Tetra Pak) cartons (e.g. juice, milk, soup cartons)",
          "✓ Clean foil and foil trays",
          "✓ Tins and cans (e.g. drink cans, food tins, biscuit or sweet tins - please rinse)",
          "✓ Empty aerosol cans (e.g. deodorant, air freshener, hairspray, de-icer)",
          "✓ Shredded paper (must be contained in a small cardboard box or envelope)",
        ];
        return array;
      }
      if (serviceName == "Domestic Waste Collection Service") {
        var array = [
          "✓ Non-recyclable rubbish",
          "✓ Nappies",
          "✓ Hygiene waste (incontinence articles, catheters etc.)",
        ];
        return array;
      }
      if (serviceName == "Garden Waste Collection Service") {
        var array = [
          "✓ Grass and hedge cuttings",
          "✓ Leaves, plants and weeds",
          "✓ Untreated wood and branches up to 100mm thick",
        ];
        return array;
      }
      if (serviceName == "Food Waste Collection Service") {
        var array = ["✓ All uneaten food and plate scrapings"];
        return array;
      }
    },
  },
};
</script>

<style scoped>
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
