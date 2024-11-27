<template>
  <div>
    <form @submit.prevent="submitForm" class="bg-light p-4 rounded shadow-sm">
      <div class="mb-3">
        <label for="amount" class="form-label">Personal id:</label>
        <input
          type="text"
          id="personalId"
          class="form-control"
          v-model="form.personalId"
          placeholder="Enter the personal id"
        />
      </div>
      <div class="mb-3">
        <label for="amount" class="form-label">Amount:</label>
        <input
          type="text"
          id="requestedAmount"
          class="form-control"
          v-model="form.requestedAmount"
          @blur="validateAmount"
          placeholder="Enter the amount"
          min="100"
          max="5000"
        />
        <div v-if="errors.amount" class="text-danger">{{ errors.amount }}</div>
      </div>

      <div class="mb-3">
        <label for="period" class="form-label">Payment Period (months):</label>
        <select
          id="period"
          class="form-select"
          v-model="form.paymentPeriod"
          @change="validatePeriod"
        >
          <option v-for="value in periods" :key="value" :value="value">
            {{ value }}
          </option>
        </select>
        <div v-if="errors.period" class="text-danger">{{ errors.period }}</div>
      </div>
      <button type="submit" class="btn btn-success w-100">Submit</button>
    </form>
    <div class="text-center mt-3">
      <b-spinner v-if="loading" variant="success" label="Spinning"></b-spinner>
      <b-container v-if="resultExists" fluid="md">
        <b-card
          border-variant="primary"
          header="Result"
          header-bg-variant="primary"
          header-text-variant="white"
          align="center"
          class="mt-3"
        >
          <b-card-text>
            <div class="mb-3">
              <b-badge v-if="result.approved" variant="success">Success</b-badge>
              <b-badge v-else variant="danger">Not approved</b-badge>
            </div>
            <div class="mb-3">
              <label for="amount" class="form-label">Approved amount: </label>
              <b>{{ result.approvedAmount }}</b>
            </div>
            <div class="mb-3">
              <label for="amount" class="form-label">Approved period: </label>
              <b>{{ result.approvedPeriod }}</b>
            </div>
          </b-card-text>
        </b-card>
      </b-container>
    </div>
  </div>

</template>

<script lang="ts">
import WebSocket from "@/components/WebSocket.vue";
import axios from "axios";
import Stomp from "webstomp-client";

export default {
  name: 'PurchaseApprovalForm',
  data() {
    return {
      result: {
        approved: false,
        approvedAmount: 0,
        approvedPeriod: 0
      },
      loading: false,
      resultExists: false,
      uuid: null,
      connected: false,
      form: {
        personalId: null,
        requestedAmount: "",
        paymentPeriod: 6,
      },
      errors: {
        amount: null,
        period: null,
      },
      periods: [6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24],
    }
  },
  methods: {
    subscribe(uuid) {
      const socket = new SockJS(import.meta.env.VITE_VUE_APP_WS_REGISTER_ENDPOINT);
      const stompClient = Stomp.over(socket);
      stompClient.connect({}, frame => {

        // Subscribe to a topic and update messages array
        stompClient.subscribe("/topic/status/" + uuid, (message) => {
          const receivedMessage = JSON.parse(message.body);
          this.loading = false;
          this.resultExists = true;
          this.result = receivedMessage;
        });
        }
      );
    },
    validateAmount() {
      const amount = this.form.requestedAmount;
      if (!amount || amount < 200 || amount > 5000 || isNaN(amount)) {
        this.errors.amount = "Amount must be between 200 and 5000.";
      } else {
        this.errors.amount = null;
      }
    },
    validatePeriod() {
      const period = this.form.paymentPeriod;
      if (!this.periods.includes(period)) {
        this.errors.period = "Please select a valid period.";
      } else {
        this.errors.period = null;
      }
    },
    validateForm() {
      this.validateAmount();
      this.validatePeriod();
      return !this.errors.amount && !this.errors.period;
    },
    async submitForm() {
      this.resultExists = false;
      this.result = {
          approved: false,
          approvedAmount: 0,
          approvedPeriod: 0
      };
      if (this.validateForm()) {
        try {
          this.loading = true;
          const response = await axios.post(import.meta.env.VITE_BACKEND_API, this.form);
          this.uuid = response.data.uuid;
          await this.subscribe(this.uuid)
        } catch (err) {
          this.error = err.message;
        }
      } else {
        alert("Please fix the errors in the form.");
      }
    },
  }
};
</script>


<style scoped>
.form-group {
  margin-bottom: 15px;
}

.error {
  color: red;
  font-size: 0.875em;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>
