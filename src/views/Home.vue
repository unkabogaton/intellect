<template>
  <v-container>
    <div>
      <v-text-field v-model.trim="url"></v-text-field>
      <v-btn @click="requestData()" :loading="loadingData">Request</v-btn>
      <div>{{ responseData }}</div>
    </div>
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
  }),
  methods: {
    // funtion for requesting data using fetch API
    async requestData() {
      if (this.url == "" || !this.url.includes("http")) {
        alert("Please type a valid url");
        return;
      }
      this.loadingData = true;
      await fetch(this.url)
        .then((response) => {
          if (response.status != 200) {
            alert("Not Found");
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
        if (typeof array[item] == "string") {
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
