<template>
  <v-container>
    <div class="mt-15">
      <div class="text-h1 text-center">
        <span class="font-weight-bold">API</span> REQUEST
      </div>
      <v-card class="pa-7 ma-7" elevation="10">
        <v-text-field
          v-model.trim="url"
          rounded
          filled
          placeholder="Type a valid link here"
          prepend-inner-icon="mdi-earth"
          @keyup="saveText"
          @keyup.enter="requestData()"
        ></v-text-field>
        <div class="text-end">
          <v-btn
            @click="requestData()"
            :loading="loadingData"
            color="primary"
            class="mx-auto"
            >Request</v-btn
          >
        </div>
      </v-card>
      <div v-if="loadingData == true" class="text-center">
        <br />
        <br />
        <v-progress-circular
          class="mt-6"
          :size="70"
          :width="7"
          indeterminate
          color="primary"
        ></v-progress-circular>
      </div>
      <div v-else>
        <v-col
          class="text-center mt-n8 ml-auto mr-2 mt-md-0 ml-md-0 mr-md-0"
          md="2"
          cols="4"
          v-show="responseData.length != 0"
        >
          <div class="caption">View</div>
          <v-select
            v-model="type"
            dense
            :items="['Card', 'Raw']"
            rounded
            filled
          >
          </v-select
        ></v-col>
        <div v-if="type == 'Card'">
          <v-card
            v-for="(response, id) in responseData"
            :key="id"
            class="pa-6 ma-2 my-6"
          >
            <v-chip class="my-2">Count of E/e: response.countE</v-chip>
            <div v-for="(value, key) in response" :key="key">
              <span class="font-weight-medium">{{ key }}</span
              >: {{ value }}
            </div>
          </v-card>
        </div>
        <v-card class="pa-7" v-else>
          <div>{{ responseData }}</div>
        </v-card>
      </div>
    </div>
    <!-- dialog box -->
    <v-dialog v-model="dialog" width="600">
      <v-card>
        <v-card-title class="headline grey lighten-2"
          >Error Occured</v-card-title
        >
        <div class="pa-5">
          <p class="font-weight-light">
            {{ promptText }}
          </p>
          <v-divider></v-divider>
        </div>
        <v-card-actions class="px-5 pb-5">
          <v-spacer></v-spacer>
          <v-btn color="grey" text @click="dialog = false"> Ok </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: "Home",
  data: () => ({
    url: "",
    responseData: [],
    loadingData: false,
    count: 0,
    promptText: "",
    dialog: false,
    type: "Card",
  }),
  created() {
    if (localStorage.getItem("url")) {
      this.url = localStorage.getItem("url");
    }
  },
  methods: {
    saveText() {
      localStorage.setItem("url", this.url);
    },
    // funtion for requesting data using fetch API
    async requestData() {
      if (this.url == "" || !this.url.includes("http")) {
        this.promptText = "Please type a valid url. Include http or https.";
        this.dialog = true;
        return;
      }
      localStorage.setItem("url", this.url);
      this.loadingData = true;
      await fetch(this.url)
        .then((response) => {
          if (response.status != 200) {
            this.promptText = "URL not found! No data was fetched.";
            this.dialog = true;
            return;
          }
          return response.json();
        })
        .then((data) => {
          // checking the type of data
          this.responseData = [];
          if (Array.isArray(data)) {
            this.reverseArray(data);
          } else {
            this.reverseObject(data);
          }
        })
        .catch((err) => {
          console.log(err);
        });
      this.loadingData = false;
    },

    // counting the number of E/e in string values
    counting(array) {
      this.count = 0;
      for (let item in array) {
        let lowerCase = array[item].toLowerCase();
        this.count += (lowerCase.match(/e/g) || []).length;
      }
    },

    // reversing and looping through the array
    reverseArray(array) {
      for (let item in array.reverse()) {
        this.reverseObject(array[item]);
      }
    },

    // reversing the contents of object by looping into the keys and values
    reverseObject(object) {
      let obj = {};
      // putting all the keys of the objects in an array then reversing each items in the array with reverseString function below
      let keys = this.reverseString(Object.keys(object));
      // putting all the values of the objects in an array then reversing each items in the array with reverseString function below
      let values = this.reverseString(Object.values(object));
      // merging/mapping the arrays into an object
      keys.forEach((element, index) => {
        obj[element] = values[index];
      });
      // adding key-value pair of the countE in the beginning of the object
      this.counting(keys);
      obj = { countE: this.count, ...obj };
      this.responseData.push(obj);
    },

    // reversing the strings in each array
    reverseString(array) {
      let reversedStrings = [];
      for (let item in array) {
        if (array[item] == "" || array[item] == null) {
          reversedStrings.push(array[item]);
        } else if (typeof array[item] == "string") {
          reversedStrings.push(array[item].split("").reverse().join(""));
        } else if (Array.isArray(array[item])) {
          reversedStrings.push(this.reverseInnerArray(array[item]));
        } else if (typeof array[item] == "object") {
          reversedStrings.push(this.reverseInnerObject(array[item]));
        } else {
          reversedStrings.push(array[item]);
        }
      }
      return reversedStrings;
    },

    // reversing the array values inside an object
    reverseInnerArray(array) {
      let reversedArray = [];
      for (let item in array.reverse()) {
        reversedArray.push(this.reverseInnerObject(array[item]));
      }
      console.log(reversedArray);
      return reversedArray;
    },

    // reversing the object values inside an arrays/objects
    reverseInnerObject(object) {
      let obj = {};
      let keys = this.reverseString(Object.keys(object));
      let values = this.reverseString(Object.values(object));
      keys.forEach((element, index) => {
        obj[element] = values[index];
      });
      return obj;
    },
  },
};
</script>
