<template>
  <div id="app">
    <h1>Cancellation Penalty Decoder</h1>
    <div class="container">
      <div class="input-group">
        <label for="textInput">JSON Input:</label>
        <textarea id="textInput" v-model="textInput" rows="4" cols="50"></textarea>
      </div>
      <p v-if="jsonError" class="error">{{ jsonError }}</p>
      <div class="input-group">
        <label for="checkIn">Check In:</label>
        <vuejs-datepicker id="checkIn" v-model="checkIn" @input="onCheckInChange"></vuejs-datepicker>
      </div>
      <div class="input-group">
        <label for="checkOut">Check Out:</label>
        <vuejs-datepicker id="checkOut" v-model="checkOut" :disabled-dates="disabledDatesForCheckOut"></vuejs-datepicker>
      </div>
      <button @click="submit">Submit</button>
    </div>
  </div>
</template>

<script>
import VuejsDatepicker from 'vuejs-datepicker';
import axios from "axios";

export default {
  name: 'App',
  components: {
    VuejsDatepicker,
  },
  data() {
    return {
      textInput: '',
      checkIn: null,
      checkOut: null,
      jsonError: null,
    };
  },
  computed: {
    disabledDatesForCheckOut() {
      if (!this.checkIn) {
        return {};
      }

      const minDate = new Date(this.checkIn.getTime() + 86400000);
      const maxDate = new Date(this.checkIn.getTime() + 500 * 86400000);

      return {
        customPredictor: date => {
          return !(date >= minDate && date <= maxDate);
        },
      };
    },
  },
  methods: {
    onCheckInChange() {
      if (this.checkOut && (this.checkOut <= this.checkIn || this.checkOut > new Date(this.checkIn.getTime() + 500 * 86400000))) {
        this.checkOut = null;
      }
    },
    validateJson() {
      try {
        JSON.parse(this.textInput);
        this.jsonError = null;
        return true;
      } catch (error) {
        this.jsonError = 'Invalid JSON format';
        return false;
      }
    },
    async submit() {
      if (this.validateJson() && this.checkIn && this.checkOut) {
        console.log('Valid JSON:', this.textInput);
        console.log('Check In:', this.checkIn);
        console.log('Check Out:', this.checkOut);

        // Send the request to the Flask API
        try {
          const ci_date = this.checkIn.toISOString().split("T")[0];
          const co_date = this.checkOut.toISOString().split("T")[0];
          const cxl_json = this.textInput;

          const response = await axios.get("http://billyleo.pythonanywhere.com/api/cxl-penalties", {
            params: {
              ci_date,
              co_date,
              cxl_json,
            },
          });
          console.log("API response:", response.data);
        } catch (error) {
          console.error("Error fetching cancellation penalties:", error);
        }
      }
    },
  },
};
</script>


<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  text-align: left;
  display: inline-block;
}

.error {
  color: red;
}
.input-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 1rem;
}

.input-group label {
  margin-bottom: 0.5rem;
}

</style>
