<template>
  <div class="container">
    <div class="form-group">
      <label>Bank: </label>
      <select id="bank" v-model="bank" class="w200" @change="reset">
        <option value="aba">ABA</option>
        <option value="ac">Acleda</option>
        <option value="wing">Wing</option>
      </select>
      <br />
      <span v-if="!bank && submitted" class="error">Required</span>
    </div>
    <div class="form-group">
      <label>Account Number: </label>
      <input
        type="number"
        v-model="bankAccountNumber"
        class="w200"
        @keyup="resetQRCode"
      />
      <br />
      <span v-if="!bankAccountNumber && submitted" class="error">Required</span>
    </div>
    <div class="form-group">
      <label>Account Name: </label>
      <input
        type="text"
        v-model="bankAccountName"
        class="w200"
        @keyup="resetQRCode"
      />
      <br>
      <span v-if="!bankAccountName && submitted" class="error">Required</span>
    </div>
    <div class="form-group">
      <label>Receive Amount: </label>
      <input type="number" v-model="amount" class="w200" @keyup="resetQRCode" />
      <br />
      <span v-if="!amount && submitted" class="error">Required</span>
      <small v-else-if="amount.length > 13" class="error">Invalid Amount</small>
    </div>
    <div class="form-group">
      <label>Select currency: </label>
      <select id="currency" v-model="currency" class="w200" @keyup="reset">
        <option value="usd">USD</option>
        <option value="kh">KH</option>
      </select>
      <br />
      <span v-if="!currency && submitted" class="error"
        >Currency is required</span
      >
    </div>
    <div class="form-group">
      <button @click="submitForm" class="submit-button">Submit</button>
    </div>
    <div class="qr-display" v-show="qrCodeData">
      <div class="logo">
        <img :src="logo" alt="logo" width="100px" />
      </div>
      <div class="khqr-info">
        <p style="margin: 0; padding-left: 25px; font-size: 12px">
          {{ bankAccountName }}
        </p>
        <p style="margin: 0; padding-left: 25px">
          {{ mapCurrnecySymbol[currency] }}
          <span style="font-size: 20px; font-weight: bold; color: black">{{
            formatAmount(amount)
          }}</span>
        </p>
      </div>
      <canvas id="canvas"></canvas>
    </div>
  </div>
</template>

<script setup>
import { BakongKHQR, khqrData, IndividualInfo } from "bakong-khqr";
import { ref } from "vue";
import QRCode from "qrcode";
import logo from "/src/asset/logo.png";

const bankAccountName = ref("");
const bankAccountNumber = ref("");
const bank = ref("aba");
const amount = ref("");
const qrCodeData = ref("");
const currency = ref("usd");
const submitted = ref(false);

const mapBakongAccountId = ref({
  aba: "abaakhppxxx@abaa",
  ac: "khqr@aclb",
  wing: "wing_khqr@wing",
});

const mapCurrnecySymbol = ref({
  usd: "USD",
  kh: "៛",
});

const mapBakongCurrency = ref({
  usd: khqrData.currency.usd,
  kh: khqrData.currency.khr,
});

const validateForm = () => {
  submitted.value = true;
  return (
    bank.value &&
    bankAccountNumber.value &&
    bankAccountName.value &&
    amount.value &&
    currency.value
  );
};

const submitForm = () => {
  if (validateForm()) {
    genreateKHQR();
  }
};

const genreateKHQR = () => {
  console.log("genreateKHQR");
  const optionalData = {
    accountInformation: bankAccountNumber.value,
    currency: mapBakongCurrency.value[currency.value],
    amount: parseFloat(amount.value),
  };
  const individualInfo = new IndividualInfo(
    mapBakongAccountId.value[bank.value],
    bankAccountName.value,
    "Phnom Penh",
    optionalData
  );
  const KHQR = new BakongKHQR();
  const individual = KHQR.generateIndividual(individualInfo);
  qrCodeData.value = individual.data.qr;

  QRCode.toCanvas(
    document.getElementById("canvas"),
    qrCodeData.value,
    { toSJISFunc: QRCode.toSJIS },
    function (error) {
      if (error) console.error(error);
      console.log("success!");
    }
  );
};

const reset = () => {
  bankAccountName.value = "";
  bankAccountNumber.value = "";
  amount.value = "";
  qrCodeData.value = "";
  submitted.value = false;
};

const resetQRCode = () => {
  qrCodeData.value = "";
  amount.value = Intl.NumberFormat('en', { maximumFractionDigits: 2 }).format(parseFloat(amount.value))
  submitted.value = false;
};

const formatAmount = (amount) => {
  console.log("amouhnt", amount);
  return new Intl.NumberFormat("en", { maximumFractionDigits: 15 }).format(
    parseFloat(amount)
  );
};
</script>

<style scoped>
.container {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  max-width: 500px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  min-height: 560px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: inline-block;
  width: 150px;
  text-align: left;
}

.w200 {
  width: 100%;
  max-width: 200px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.submit-button {
  background-color: #42b983;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.submit-button:hover {
  background-color: #358a65;
}

.error {
  color: red;
  font-size: 0.8em;
  margin-left: 10px;
}

.logo {
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  width: 100%;
  background: #e0252e;
  position: relative;
}

.logo::after {
  position: absolute;
  background-color: #e0252e;
  content: " ";
  width: 40px;
  height: 20px;
  transform: rotate(45deg);
  top: 35px;
  right: -15px;
}

.qr-display {
  border-radius: 20px;
  width: 100%;
  max-width: 200px;
  margin: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0px 0 20px black;
  overflow: hidden;
}

.khqr-info {
  margin-top: 10px;
  width: 100%;
  text-align: left;
  padding-bottom: 8px;
  border-bottom: 1px dashed;
}

.khqr-info h3 {
  margin: 0px;
  padding: 0px 25px;
}

#canvas {
  padding-bottom: 10px;
}

#app {
  height: 100vh;
  display: flex;
  align-items: center;
}

@media (max-width: 600px) {
  .container {
    padding: 10px;
    min-height: auto;
    box-shadow: none;
  }

  label {
    width: 100%;
    text-align: left;
    margin-bottom: 5px;
  }

  .form-group {
    text-align: left;
  }

  .w200 {
    max-width: 100%;
  }

  .submit-button {
    width: 100%;
  }

  .qr-display {
    width: 100%;
  }
}
</style>
