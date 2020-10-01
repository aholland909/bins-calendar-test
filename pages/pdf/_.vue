<template>
  <div class="container">
    <div class="binform-pdf" id="pdf">
      <div class="binform-pdf-key">
        <div class="key-icons">
          Key:
        </div>
        <div class="key-icons">
          Food waste
          <b-icon
            icon="checkbox-blank-circle"
            size="is-small"
            class="material-icons"
            type="is-twitter"
          ></b-icon>
        </div>
        <div class="key-icons">
          Recycling
          <b-icon
            icon="checkbox-blank-circle"
            size="is-small"
            class="material-icons"
            type="is-recycling"
          ></b-icon>
        </div>
        <div class="key-icons">
          Rubbish
          <b-icon
            icon="checkbox-blank-circle"
            size="is-small"
            class="material-icons"
            type="is-rubbish"
          ></b-icon>
        </div>
        <div class="key-icons">
          Garden
          <b-icon
            icon="checkbox-blank-circle"
            size="is-small"
            class="material-icons"
            type="is-garden"
          ></b-icon>
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
          >
            <template slot="header">
              <span class>{{ monthNames[(new Date().getMonth() + n-1) % 12] }}</span>
            </template>
          </b-datepicker>
        </div>
      </div>
      <!-- <div v-for="c in collections" :key="c.id">
        <p style="font-size: 0.8rem;font-weight: 300;">{{c}}</p>
      </div>-->
    </div>
  </div>
</template>

<script>
import axios from "axios";
import debounce from "lodash/debounce";
import defer from "promise-defer";
// import * as jsPDF from "jspdf";

// yarn add promise-defer
export default {
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
      path: this.$route.params.pathMatch,
      uprn: "",
      postcode: "",
      collections: [],
      collectionsOld: [
        {
          Date: "01/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "08/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "15/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "22/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "29/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "29/10/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "05/11/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "12/11/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "19/11/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "26/11/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "26/11/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "03/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "10/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "17/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "24/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "24/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "31/12/2020 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "07/01/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "14/01/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "21/01/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "21/01/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "28/01/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "04/02/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "11/02/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "18/02/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "18/02/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "25/02/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "04/03/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "11/03/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "18/03/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "18/03/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "25/03/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "01/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "08/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "15/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "15/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "22/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "29/04/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "06/05/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "13/05/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "13/05/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "20/05/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "27/05/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "03/06/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "10/06/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "10/06/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "17/06/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "24/06/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "01/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "08/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "08/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "15/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "22/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "29/07/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "05/08/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "05/08/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "12/08/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "19/08/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "26/08/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "02/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "02/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "09/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "16/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "23/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "30/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "30/09/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "07/10/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "14/10/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },

        {
          Date: "21/10/2021 00:00:00",
          Day: "Thursday",
          Service: "Recycling Collection Service",
        },

        {
          Date: "28/10/2021 00:00:00",
          Day: "Thursday",
          Service: "Domestic Waste Collection Service",
        },
      ],
      collectionDisplay: [],
    };
  },
  mounted() {
    var pathArray = this.path.split("/");
    this.uprn = pathArray[0];
    this.postcode = pathArray[1];
    // this.yearOutput();
  },

  methods: {
    pdfgencss() {
      console.log("test pdf output with css");
      if (process.browser) {
        window.print();
      }
    },
    yearOutput() {
      // const loadingComponent = this.$buefy.loading.open({
      //   // container: this.$refs.element
      // });
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
          loadingComponent.close();
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

<style>
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
  background-color: lightgray;
  width: 210mm;
  height: 287mm;
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
  padding: 0.3rem 0.5rem;
}
.datepicker .datepicker-table .datepicker-body.has-events .datepicker-cell {
  padding: 0.3rem 0.5rem 0.6rem;
}
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