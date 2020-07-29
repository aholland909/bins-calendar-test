<template>
  <div class="container">
    <div class="binform">
      <h1 class="subtitle">When is my bin collected?</h1>
      <b-field class="mainsearch">
        <b-autocomplete
          class="searchbox"
          size="is-large"
          v-model="name"
          ref="autocomplete"
          :data="data"
          :loading="isFetching"
          :open-on-focus="false"
          :clear-on-select="false"
          placeholder="e.g. RG1 2LU"
          icon="earth"
          @typing="getAsyncData"
          @select="selectAddress"
        >
          <template slot-scope="props">
            <div class="media">
              <div class="media-content search-result"><p>{{props.option.SiteShortAddress}}</p></div>
            </div>
          </template>

          <template slot="empty">No results for {{name}}</template>
          <template slot="footer">
        
              <p>
            <a @click="addressFindError">
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

      <!-- <div v-if="selected">{{selected}}</div>
      <br />
      <div v-if="collections.length">{{collections}}</div>-->

      <div v-if="collectionsError" class="tile is-child box">
        <p>No kerbside collection for {{selected.SiteShortAddress}}.</p>
        <p>If this property is a flat there will be an alternative collection arrangement.</p>
      </div>

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
                  <b-icon :icon="props.open ? 'menu-down' : 'menu-up'"></b-icon>
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

export default {
  data() {
    return {
      name: "",
      data: [
        [
          {
            AccountSiteId: "33602",
            AccountSiteUprn: "310014179",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467327.0",
            SiteId: "119",
            SiteLatitude: "51.4588861255",
            SiteLongitude: "-1.03232862226",
            SiteNorthing: "173769.0",
            SiteShortAddress: "1, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "41554",
            AccountSiteUprn: "310014184",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467327.0",
            SiteId: "30784",
            SiteLatitude: "51.4588861255",
            SiteLongitude: "-1.03232862226",
            SiteNorthing: "173769.0",
            SiteShortAddress: "3, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "81369",
            AccountSiteUprn: "310028424",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467332.0",
            SiteId: "45569",
            SiteLatitude: "51.4587776493",
            SiteLongitude: "-1.03225894749",
            SiteNorthing: "173757.0",
            SiteShortAddress: "5, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "34138",
            AccountSiteUprn: "310019831",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467330.0",
            SiteId: "54891",
            SiteLatitude: "51.4587509161",
            SiteLongitude: "-1.03228829997",
            SiteNorthing: "173754.0",
            SiteShortAddress: "7, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "19105",
            AccountSiteUprn: "310012860",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467327.0",
            SiteId: "61850",
            SiteLatitude: "51.4586793507",
            SiteLongitude: "-1.03233299386",
            SiteNorthing: "173746.0",
            SiteShortAddress: "9, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "12715",
            AccountSiteUprn: "310007971",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467326.0",
            SiteId: "4219",
            SiteLatitude: "51.4586435086",
            SiteLongitude: "-1.03234814521",
            SiteNorthing: "173742.0",
            SiteShortAddress: "11, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "36641",
            AccountSiteUprn: "310003124",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467325.0",
            SiteId: "8606",
            SiteLatitude: "51.4585717056",
            SiteLongitude: "-1.03236405678",
            SiteNorthing: "173734.0",
            SiteShortAddress: "13, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "62393",
            AccountSiteUprn: "310056833",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467323.0",
            SiteId: "11724",
            SiteLatitude: "51.4585359822",
            SiteLongitude: "-1.03239359912",
            SiteNorthing: "173730.0",
            SiteShortAddress: "15, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "41216",
            AccountSiteUprn: "310002069",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467324.0",
            SiteId: "14928",
            SiteLatitude: "51.4584279809",
            SiteLongitude: "-1.03238148877",
            SiteNorthing: "173718.0",
            SiteShortAddress: "17, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "24891",
            AccountSiteUprn: "310029502",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467316.0",
            SiteId: "17640",
            SiteLatitude: "51.458357009",
            SiteLongitude: "-1.03249813714",
            SiteNorthing: "173710.0",
            SiteShortAddress: "19, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "36191",
            AccountSiteUprn: "310029099",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467316.0",
            SiteId: "21845",
            SiteLatitude: "51.458312058",
            SiteLongitude: "-1.03249908731",
            SiteNorthing: "173705.0",
            SiteShortAddress: "21, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "75471",
            AccountSiteUprn: "310059607",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467291.0",
            SiteId: "22901",
            SiteLatitude: "51.4583330058",
            SiteLongitude: "-1.03285848183",
            SiteNorthing: "173707.0",
            SiteShortAddress: "22, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "52326",
            AccountSiteUprn: "310024506",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467315.0",
            SiteId: "23982",
            SiteLatitude: "51.4582762158",
            SiteLongitude: "-1.0325142384",
            SiteNorthing: "173701.0",
            SiteShortAddress: "23, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "50588",
            AccountSiteUprn: "310061435",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467276.0",
            SiteId: "25076",
            SiteLatitude: "51.4580470988",
            SiteLongitude: "-1.033080424",
            SiteNorthing: "173675.0",
            SiteShortAddress: "24, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "6360",
            AccountSiteUprn: "310008749",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467315.0",
            SiteId: "26141",
            SiteLatitude: "51.4582312648",
            SiteLongitude: "-1.03251518855",
            SiteNorthing: "173696.0",
            SiteShortAddress: "25, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "55592",
            AccountSiteUprn: "310048902",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467275.0",
            SiteId: "27127",
            SiteLatitude: "51.4579932761",
            SiteLongitude: "-1.03309595438",
            SiteNorthing: "173669.0",
            SiteShortAddress: "26, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "76324",
            AccountSiteUprn: "310011796",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467313.0",
            SiteId: "28025",
            SiteLatitude: "51.4581505903",
            SiteLongitude: "-1.03254568066",
            SiteNorthing: "173687.0",
            SiteShortAddress: "27, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "73364",
            AccountSiteUprn: "310003721",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467309.0",
            SiteId: "29796",
            SiteLatitude: "51.4580971238",
            SiteLongitude: "-1.03260438444",
            SiteNorthing: "173681.0",
            SiteShortAddress: "29, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "16866",
            AccountSiteUprn: "310061615",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467308.0",
            SiteId: "32995",
            SiteLatitude: "51.4580612817",
            SiteLongitude: "-1.03261953538",
            SiteNorthing: "173677.0",
            SiteShortAddress: "31, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "59683",
            AccountSiteUprn: "310000564",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467313.0",
            SiteId: "34516",
            SiteLatitude: "51.457970786",
            SiteLongitude: "-1.0325494811",
            SiteNorthing: "173667.0",
            SiteShortAddress: "33, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "46466",
            AccountSiteUprn: "310006010",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467309.0",
            SiteId: "35992",
            SiteLatitude: "51.4578993391",
            SiteLongitude: "-1.03260856465",
            SiteNorthing: "173659.0",
            SiteShortAddress: "35, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "26359",
            AccountSiteUprn: "310001951",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467308.0",
            SiteId: "37285",
            SiteLatitude: "51.4578455165",
            SiteLongitude: "-1.03262409553",
            SiteNorthing: "173653.0",
            SiteShortAddress: "37, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "67273",
            AccountSiteUprn: "310051168",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467305.0",
            SiteId: "38540",
            SiteLatitude: "51.4577469803",
            SiteLongitude: "-1.03266935799",
            SiteNorthing: "173642.0",
            SiteShortAddress: "39, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "16108",
            AccountSiteUprn: "310041991",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467303.0",
            SiteId: "41117",
            SiteLatitude: "51.4576932764",
            SiteLongitude: "-1.03269927952",
            SiteNorthing: "173636.0",
            SiteShortAddress: "41, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "20203",
            AccountSiteUprn: "310032663",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467301.0",
            SiteId: "42266",
            SiteLatitude: "51.4576215921",
            SiteLongitude: "-1.03272958096",
            SiteNorthing: "173628.0",
            SiteShortAddress: "43, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "72980",
            AccountSiteUprn: "310011671",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467297.0",
            SiteId: "43224",
            SiteLatitude: "51.4575501451",
            SiteLongitude: "-1.03278866377",
            SiteNorthing: "173620.0",
            SiteShortAddress: "45, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "65878",
            AccountSiteUprn: "310007323",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467294.0",
            SiteId: "44175",
            SiteLatitude: "51.4574336283",
            SiteLongitude: "-1.03283430549",
            SiteNorthing: "173607.0",
            SiteShortAddress: "47, BLUNDELLS ROAD, RG30 4TR",
          },
          {
            AccountSiteId: "7703",
            AccountSiteUprn: "310006774",
            SiteAddress2: "BLUNDELLS ROAD",
            SiteAddressPrefix: "None",
            SiteEasting: "467293.0",
            SiteId: "45052",
            SiteLatitude: "51.4573438448",
            SiteLongitude: "-1.03285059577",
            SiteNorthing: "173597.0",
            SiteShortAddress: "49, BLUNDELLS ROAD, RG30 4TR",
          },
        ],
      ],
      results: [],
      selected: {
        AccountSiteId: "34138",
        AccountSiteUprn: "310019831",
        SiteAddress2: "BLUNDELLS ROAD",
        SiteAddressPrefix: "None",
        SiteEasting: "467330.0",
        SiteId: "54891",
        SiteLatitude: "51.4587509161",
        SiteLongitude: "-1.03228829997",
        SiteNorthing: "173754.0",
        SiteShortAddress: "7, BLUNDELLS ROAD, RG30 4TR",
      },
      isFetching: false,
      isExpanded: false,
      addresses: [],
      collections: [
        {
          Date: "30/07/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "06/08/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "13/08/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "20/08/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },
      ],
      collectionsError: false,
      // wp: process.env.WP_URL,
      // base_url: process.env.BASE_URL,
    };
  },
  methods: {
    getAsyncData: debounce(function (name) {
      if (!name.length) {
        this.data = [];
        return;
      }
      if (this.valid_postcode(name)) {
        this.isFetching = true;
        axios
          .get(
            "https://cors-anywhere.herokuapp.com/" +
              "http://3.9.226.211/" +
              "rbc/getaddresses/" +
              name
          )
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
        .get(
          "https://cors-anywhere.herokuapp.com/" +
            "http://3.9.226.211/" +
            "rbc/mycollections/" +
            this.selected.AccountSiteUprn
        )
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
        message: "Check who your local authority is https://www.gov.uk/find-local-council",
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
        var array = [
          "✓ All uneaten food and plate scrapings",
          "✓ Cheese",
          "✓ Eggs",
          "✓ Bread",
          "✓ Cakes",
          "✓ Pastries",
          "✓ Raw meat",
          "✓ Cooked meat",
          "✓ Meat bones",
          "✓ Tea bags",
          "✓ Coffee grounds",
          "✓ Raw vegetables",
          "✓ Cooked vegetables",
          "✓ Whole fruit",
          "✓ Fruit and vegatable peelings",
          "✓ Raw fish",
];
        return array;
      }
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