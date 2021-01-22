<template>
  <div class="book" v-if="!isLoading">
    <div class="page">
      <div class="subpage">
        <div class="binform-container">
          <div class="binform-banner">
            <div class="binform-banner-text">
              <div>Waste Collection Calendar</div>
            </div>
            <!-- <div class="binform-banner-round">A</div> -->
          </div>
          <div class="binform-address">{{ this.address }}</div>
          <div class="binform-info-banner">
            <div class="binform-info-text">
              Your latest collection dates can be found at:
              <a href="https://www.reading.gov.uk/bin-calendar/"
                >www.reading.gov.uk/bin-calendar/</a
              >
            </div>
            <div class="binform-key">
              <div v-for="n in 4" :key="n">
                <img
                  class="bin-key-image"
                  :src="require(`~/assets/icons/binKey/binimage${n}.jpg`)"
                />
              </div>
            </div>
          </div>
          <div class="binform-reminder">
            For a reminder of what you can put in each bin please visit:
            <a href="https://www.reading.gov.uk/recycling/"
              >www.reading.gov.uk/recycling/</a
            >
          </div>
          <div class="bin-cal-container">
            <div
              class="bin-cal"
              v-for="(collectionArray, month) in groupedCollectionsTempA"
              :key="month"
            >
              <div class="bin-cal-heading">{{ month }}</div>
              <div
                class="bin-cal-item"
                v-for="(collections, n) in collectionArray"
                :key="n"
              >
                <div class="bin-cal-item-date">
                  {{ formatCollectionDate(Object.keys(collections)[0]) }}
                </div>
                <div class="bin-cal-item-images-container">
                  <div
                    role="img"
                    v-for="(service, index) in mapOrder(
                      collections[Object.keys(collections)[0]],
                      item_order,
                      'Service'
                    )"
                    :key="index"
                    :aria-label="service.Service"
                  >
                    <img
                      class="bin-image"
                      :src="getImage(service.Service)"
                      fill="white"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="binform-footer">
            <img src="~/assets/images/bin-cal-footer.jpg" />
          </div>
        </div>
      </div>
    </div>
    <div class="page">
      <div class="subpage">
        <div class="binform-container">
          <div class="binform-banner">
            <div class="binform-banner-text">
              <div>Waste Collection Calendar</div>
            </div>
            <!-- <div class="binform-banner-round">A</div> -->
          </div>
          <div class="binform-info-banner">
            <div class="binform-info-text">
              Your latest collection dates can be found at:
              <a href="https://www.reading.gov.uk/bin-calendar/"
                >www.reading.gov.uk/bin-calendar/</a
              >
            </div>
            <div class="binform-key">
              <div v-for="n in 4" :key="n">
                <img
                  class="bin-key-image"
                  :src="require(`~/assets/icons/binKey/binimage${n}.jpg`)"
                />
              </div>
            </div>
          </div>
          <div class="binform-reminder">
            For a reminder of what you can put in each bin please visit:
            <a href="https://www.reading.gov.uk/recycling/"
              >www.reading.gov.uk/recycling/</a
            >
          </div>
          <div class="bin-cal-container">
            <div
              class="bin-cal"
              v-for="(collectionArray, month) in groupedCollectionsTempB"
              :key="month"
            >
              <div class="bin-cal-heading">{{ month }}</div>
              <div
                class="bin-cal-item"
                v-for="(collections, n) in collectionArray"
                :key="n"
              >
                <div class="bin-cal-item-date">
                  {{ formatCollectionDate(Object.keys(collections)[0]) }}
                </div>
                <div class="bin-cal-item-images-container">
                  <div
                    role="img"
                    v-for="(service, index) in mapOrder(
                      collections[Object.keys(collections)[0]],
                      item_order,
                      'Service'
                    )"
                    :key="index"
                    :aria-label="service.Service"
                  >
                    <img
                      class="bin-image"
                      :src="getImage(service.Service)"
                      fill="white"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="binform-footer">
            <img src="~/assets/images/bin-cal-footer-two.jpg" />
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
  props: {
    uprn: {
      type: String,
      required: false,
      default: "",
    },
    address: {
      type: String,
      required: false,
      default: "",
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
      days: [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ],
      item_order: [
        "Domestic Waste Collection Service",
        "Recycling Collection Service",
        "Garden Waste Collection Service",
        "Food Waste Collection Service",
      ],

      collections: [],
      collectionsTemp: [
        {
          Date: "03/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "03/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Garden Waste Collection Service",
        },
        {
          Date: "03/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "10/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "10/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "17/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "17/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Garden Waste Collection Service",
        },
        {
          Date: "17/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "24/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "24/02/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "03/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "03/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Garden Waste Collection Service",
        },
        {
          Date: "03/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "10/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "10/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "17/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "17/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Garden Waste Collection Service",
        },
        {
          Date: "17/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "24/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "24/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "31/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "31/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Garden Waste Collection Service",
        },
        {
          Date: "31/03/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "07/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "07/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "14/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "14/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "21/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "21/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "28/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "28/04/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "05/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "05/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "12/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "12/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "19/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "19/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "26/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "26/05/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "02/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "02/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "09/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "09/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "16/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "16/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "23/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "23/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "30/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "30/06/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "07/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "07/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "14/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "14/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "21/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "21/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "28/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "28/07/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "04/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "04/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "11/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "11/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "18/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "18/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "25/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "25/08/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "01/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "01/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "08/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "08/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "15/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "15/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "22/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "22/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "29/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "29/09/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "06/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "06/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "13/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "13/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "20/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "20/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "27/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "27/10/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "03/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "03/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "10/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "10/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "17/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "17/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "24/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "24/11/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "01/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "01/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "08/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "08/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "15/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "15/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "22/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "22/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "29/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "29/12/2021 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "05/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "05/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "12/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "12/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "19/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
        {
          Date: "19/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Recycling Collection Service",
        },
        {
          Date: "26/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Domestic Waste Collection Service",
        },
        {
          Date: "26/01/2022 00:00:00",
          Day: "Wednesday",
          Service: "Food Waste Collection Service",
        },
      ],
      groupedCollections: [],
      groupedCollectionsTempA: {},
      groupedCollectionsTempB: {},
      groupedCollectionsTemp: {
        "October 2020": [
          {
            "Wed Oct 07 2020 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "07/10/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
              {
                Date: "07/10/2020 00:00:00",
                Day: "Wednesday",
                Service: "Food Waste Collection Service",
              },
            ],
          },
          {
            "Thu Oct 08 2020 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "08/10/2020 00:00:00",
                Day: "Thursday",
                Service: "Food Waste Collection Service",
              },
              {
                Date: "08/10/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Oct 15 2020 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "15/10/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Oct 21 2020 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "21/10/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Oct 22 2020 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "22/10/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Oct 29 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "29/10/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "November 2020": [
          {
            "Wed Nov 04 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "04/11/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Nov 05 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "05/11/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Nov 12 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "12/11/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Nov 18 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "18/11/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Nov 19 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "19/11/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Nov 26 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "26/11/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "December 2020": [
          {
            "Wed Dec 02 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "02/12/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Dec 03 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "03/12/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Dec 10 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "10/12/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Dec 16 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "16/12/2020 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Dec 17 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "17/12/2020 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Dec 24 2020 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "24/12/2020 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "January 2021": [
          {
            "Sat Jan 02 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "02/01/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Mon Jan 04 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "04/01/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Sat Jan 09 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "09/01/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Jan 14 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "14/01/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Fri Jan 15 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "15/01/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Jan 21 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "21/01/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Jan 27 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "27/01/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Jan 28 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "28/01/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
        ],
        "February 2021": [
          {
            "Thu Feb 04 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "04/02/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Feb 10 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "10/02/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Feb 11 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "11/02/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Feb 18 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "18/02/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Feb 24 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "24/02/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Feb 25 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "25/02/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
        ],
        "March 2021": [
          {
            "Thu Mar 04 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "04/03/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Mar 10 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "10/03/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Mar 11 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "11/03/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Mar 18 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "18/03/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Wed Mar 24 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "24/03/2021 00:00:00",
                Day: "Wednesday",
                Service: "Garden Waste Collection Service",
              },
            ],
          },
          {
            "Thu Mar 25 2021 00:00:00 GMT+0000 (Greenwich Mean Time)": [
              {
                Date: "25/03/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
        ],
        "April 2021": [
          {
            "Thu Apr 01 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "01/04/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Apr 08 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "08/04/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Apr 15 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "15/04/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Apr 22 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "22/04/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Apr 29 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "29/04/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "May 2021": [
          {
            "Thu May 06 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "06/05/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu May 13 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "13/05/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu May 20 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "20/05/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu May 27 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "27/05/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "June 2021": [
          {
            "Thu Jun 03 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "03/06/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Jun 10 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "10/06/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Jun 17 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "17/06/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Jun 24 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "24/06/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
        "July 2021": [
          {
            "Thu Jul 01 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "01/07/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Jul 08 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "08/07/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Jul 15 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "15/07/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Jul 22 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "22/07/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Jul 29 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "29/07/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
        ],
        "August 2021": [
          {
            "Thu Aug 05 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "05/08/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Aug 12 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "12/08/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Aug 19 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "19/08/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Aug 26 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "26/08/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
        ],
        "September 2021": [
          {
            "Thu Sep 02 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "02/09/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Sep 09 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "09/09/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Sep 16 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "16/09/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
          {
            "Thu Sep 23 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "23/09/2021 00:00:00",
                Day: "Thursday",
                Service: "Recycling Collection Service",
              },
            ],
          },
          {
            "Thu Sep 30 2021 00:00:00 GMT+0100 (British Summer Time)": [
              {
                Date: "30/09/2021 00:00:00",
                Day: "Thursday",
                Service: "Domestic Waste Collection Service",
              },
            ],
          },
        ],
      },
      collectionDisplay: [],
      newCollections: [],
      rbcapi: process.env.rbcapi,
    };
  },

  mounted() {
    //start getcollections
    // console.log(this.groupedCollectionsTemp);
    this.yearOutput(); //disable for testing
    // this.yearOutputTesting(); //enable for testing
  },

  methods: {
    formatTitle(datetime) {
      var cal = new Date(datetime);
      return this.monthNames[cal.getMonth()] + " " + cal.getFullYear();
    },
    formatCollectionDate(datetime) {
      var cal = new Date(datetime);
      return (
        this.days[cal.getDay()] +
        " " +
        cal.getDate() +
        this.FormatDaySuffix(cal.getDate())
      );
    },
    formatTitleFromIndex(m) {
      var cal = new Date();
      cal.setMonth(cal.getMonth() + m);
      return this.monthNames[cal.getMonth()] + " " + cal.getFullYear();
    },
    //group collections with the same date
    groupByDay(objectArray, property) {
      return objectArray.reduce(function (acc, obj) {
        var reggie = /(\d{2})\/(\d{2})\/(\d{4}) (\d{2}):(\d{2}):(\d{2})/;
        var dateArray = reggie.exec(obj[property]);
        var dateObject = new Date(
          +dateArray[3],
          +dateArray[2] - 1, // Careful, month starts at 0!
          +dateArray[1],
          +dateArray[4],
          +dateArray[5],
          +dateArray[6]
        );

        let key = dateObject;

        if (!acc[key]) {
          acc[key] = [];
        }
        acc[key].push(obj);
        return acc;
      }, {});
    },
    //group collections with the same month
    groupByMonth(jsonObj) {
      return Object.keys(jsonObj).reduce((prev, key, i, all) => {
        let monthArr = [];
        if (typeof prev === "string") {
          var obj = [];
          var temp = { [prev]: jsonObj[prev] };
          obj.push(temp);
          monthArr[this.formatTitle(prev)] = obj;
          prev = {};
        }
        var monthName = this.formatTitle(key);
        if (prev[monthName]) {
          var obj = [];
          var temp = { [key]: jsonObj[key] };
          prev[monthName].push(temp);
        } else if (monthArr[monthName]) {
          var obj = [];
          var temp = { [key]: jsonObj[key] };
          monthArr[monthName].push(temp);
        } else {
          var obj = [];
          var temp = { [key]: jsonObj[key] };
          obj.push(temp);
          monthArr[monthName] = obj;
        }
        return { ...prev, ...monthArr };
      });
    },
    yearOutputTesting() {
      // this.collectionsTemp temp array

      var groupedtemp = this.groupByDay(this.collectionsTemp, "Date");
      // console.log(groupedtemp);
      this.groupedCollections = this.groupByMonth(groupedtemp);
      // console.log(this.groupedCollections);
      var keys = Object.keys(this.groupedCollections);
      var keysA = keys.slice(0, keys.length / 2);
      var keysB = keys.slice(keys.length / 2, keys.length);

      keysA.forEach((key0) => {
        this.groupedCollectionsTempA[key0] = this.groupedCollections[key0];
      });

      keysB.forEach((key) => {
        this.groupedCollectionsTempB[key] = this.groupedCollections[key];
      });

      this.isLoading = false;
      this.$emit("domRendered");
    },
    yearOutput() {
      //first day of month
      var today = new Date();
      today.setDate(1);
      today.setMonth(new Date().getMonth());
      today.setHours(0, 0, 0, 0);
      var todayplusyear = new Date();
      todayplusyear.setDate(1);
      todayplusyear.setHours(0, 0, 0, 0);
      todayplusyear.setMonth(new Date().getMonth());
      todayplusyear.setFullYear(today.getFullYear() + 1);
      console.log(today);
      console.log(todayplusyear);
      this.collections = [];
      if (this.uprn != null && this.uprn.length == 9) {
        this.getCollectionByDate(
          this.uprn,
          today,
          todayplusyear,
          this.collections
        ).then(() => {
          // update other cal
          console.log(this.collections);
          var groupedtemp = this.groupByDay(this.collections, "Date");
          // console.log(groupedtemp);
          this.groupedCollections = this.groupByMonth(groupedtemp);
          // console.log(this.groupedCollections);
          var keys = Object.keys(this.groupedCollections);
          var keysA = keys.slice(0, keys.length / 2);
          var keysB = keys.slice(keys.length / 2, keys.length);

          keysA.forEach((key0) => {
            this.groupedCollectionsTempA[key0] = this.groupedCollections[key0];
          });

          keysB.forEach((key) => {
            this.groupedCollectionsTempB[key] = this.groupedCollections[key];
          });

          this.isLoading = false;
          this.$emit("domRendered");
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
          this.rbcapi + "rbc/mycollections/" + uprn + "/" + formattedstartDate
        )
        .then((response) => {
          // handle success
          if (response.data.Collections) {
            if (new Date(startDate) < new Date(endDate)) {
              response.data.Collections.forEach((item) => {
                //check each element for start date and end date diff
                if (
                  new Date(this.dateFormatter(item.Date)) < new Date(endDate)
                ) {
                  collectionsArray.push(item);
                }
              });
              //add one day to new start date
              var oldStartDate = this.dateFormatter(
                response.data.Collections[response.data.Collections.length - 1]
                  .Date
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
    getImage(serviceName) {
      if (serviceName == "Recycling Collection Service") {
        return require("~/assets/icons/" + "redWaste.png");
      }
      if (serviceName == "Domestic Waste Collection Service") {
        return require("~/assets/icons/" + "greyWaste.png");
      }
      if (serviceName == "Garden Waste Collection Service") {
        return require("~/assets/icons/" + "greenWaste.png");
      }
      if (serviceName == "Food Waste Collection Service") {
        return require("~/assets/icons/" + "foodWaste.png");
      }
    },
    dateFormatter(dateString) {
      var reggie = /(\d{2})\/(\d{2})\/(\d{4}) (\d{2}):(\d{2}):(\d{2})/;
      var dateArray = reggie.exec(dateString);
      // console.log(dateArray);
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
    FormatDaySuffix(day) {
      let lastDigit = day % 10; //get last digit
      let tens = Math.floor(day / 10); //get tens digit

      if (tens !== 1) {
        if (lastDigit == 1) {
          return "st";
        } else if (lastDigit == 2) {
          return "nd";
        } else if (lastDigit == 3) {
          return "rd";
        } else {
          return "th";
        }
      } else {
        // 10-19th
        return "th";
      }
    },
    mapOrder(array, order, key) {
      return array.slice().sort(function (a, b) {
        var A = a[key],
          B = b[key];

        if (order.indexOf(A) > order.indexOf(B)) {
          return 1;
        } else {
          return -1;
        }
      });
    },
    serviceNameEdit(service) {
      return service.replace(/Collection Service/g, "");
    },
  },
};
</script>