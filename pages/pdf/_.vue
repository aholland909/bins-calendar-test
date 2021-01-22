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
      path: this.$route.params.pathMatch,
      uprn: "",
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
    this.address = pathArray[1];
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
    }
  },
};
</script>