<template>
  <div class="container">
    <div class="binform">
      <h1 class="subtitle">Bin test calendar</h1>
      <b-field class="mainsearch">
        <b-autocomplete
          style="display:inline-block;width:50%;"
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

      <!-- <div v-if="selected">{{selected}}</div>
      <br />
      <div v-if="collections.length">{{collections}}</div>-->

      <div v-if="collections.length">
        <div v-for="c in collections" :key="c">
          <div class="tile is-ancestor">
            <div class="tile is-vertical is-parent">
              <div class="tile is-child box">
                <p class="title is-2">{{c.Service}}</p>
                <!-- <p>{{c.Day}}</p> -->
                <p>{{c.Date}}</p>
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

export default {
  data() {
    return {
      name: "",
      data: [],
      results: [],
      selected: null,
      isFetching: false,
      isExpanded: false,
      addresses: [],
      collections: []
      // wp: process.env.WP_URL,
      // base_url: process.env.BASE_URL,
    };
  },
  computed: {
    async filteredDataArray() {}
  },
  methods: {
    getAsyncData: debounce(function(name) {
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
              data.Addresses.forEach(item => this.data.push(item));

              //sort addresses
              this.data = this.data.sort(this.compareAddress);
            }
          })
          .catch(error => {
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
      this.selected = selected;
      axios
        .get(
          "https://cors-anywhere.herokuapp.com/" +
            "http://3.9.226.211/" +
            "/rbc/mycollections/" +
            this.selected.AccountSiteUprn
        )
        .then(({ data }) => {
          this.collections = [];

          data.Collections.forEach(item => this.collections.push(item));
        })
        .catch(error => {
          this.data = [];
          throw error;
        });        
    },
    formatDate(date) {
      var d = new Date(date);

      return;
    }
  }
};
</script>

<style>
.mainsearch {
  display: flex !important;
  justify-content: center !important;
}
.binform {
  padding-top: 2rem;
  width: 100%;
}
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
