<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import axios from "axios";

const router = useRouter();
const route = useRoute();
const currentStep = ref(1);
const totalSteps = 3;
const isEditMode = ref(false);

if (!localStorage.getItem("token")) {
  router.push("/");
}

// Step 1: Invoice Details
const invoiceDetails = ref({
  customerName: "",
  customerPhone: "",
  customerEmail: "",
  customerGSTIN: "",
  customerAddress: "",
  paymentDate: "",
  paymentMode: "",
  invoiceDate: new Date().toISOString().substr(0, 10),
  recipientName: "",
  recipientPhone: "",
  recipientEmail: "",
  recipientPincode: "",
  recipientCity: "",
  recipientState: "",
  recipientAddress: "",
});

// Step 2: Transaction Details
const items = ref([
  {
    id: 1,
    product: "Note Book (200 pages)",
    rate: 120,
    quantity: 20,
    discount: 200,
    total: 2400,
  },
  { id: 2, product: "", rate: 0, quantity: 0, discount: 0, total: 0 },
]);

// Validation rules
const nameRules = [
  (v: string) => !!v || "Name is required",
  (v: string) =>
    /^[a-zA-Z\s]+$/.test(v) ||
    "Name cannot contain special characters or numbers",
];
const phoneRules = [
  (v: string) => !!v || "Phone number is required",
  (v: string) => /^\d{10}$/.test(v) || "Phone number must be 10 digits",
];
const emailRules = [
  (v: string) => !!v || "Email is required",
  (v: string) => /.+@.+\..+/.test(v) || "Email must be valid",
];
const addressRules = [(v: string) => !!v || "Address is required"];
const gstinRules = [(v: string) => !!v || "GSTIN is required"];
const pincodeRules = [
  (v: string) => !!v || "Pincode is required",
  (v: string) => /^\d{6}$/.test(v) || "Pincode must be 6 digits",
];
const cityRules = [(v: string) => !!v || "City is required"];
const stateRules = [(v: string) => !!v || "State is required"];
const productRules = [(v: string) => !!v || "Product is required"];
const rateRules = [(v: number) => v > 0 || "Rate must be greater than 0"];
const quantityRules = [
  (v: number) => v > 0 || "Quantity must be greater than 0",
];

// Form validation states
const step1Valid = ref(false);
const step2Valid = ref(false);

// Calculate totals
const calculateItemTotal = (item: any) => {
  return item.rate * item.quantity - item.discount;
};

const updateItemTotal = (item: any) => {
  item.total = calculateItemTotal(item);
};

const addItem = () => {
  const newId =
    items.value.length > 0
      ? Math.max(...items.value.map((item) => item.id)) + 1
      : 1;
  items.value.push({
    id: newId,
    product: "",
    rate: 0,
    quantity: 0,
    discount: 0,
    total: 0,
  });
};

const removeItem = (id: number) => {
  if (items.value.length > 1) {
    items.value = items.value.filter((item) => item.id !== id);
  }
};

const total = computed(() => {
  return items.value.reduce((sum, item) => sum + item.total, 0);
});

// Navigation functions
const nextStep = () => {
  if (currentStep.value < totalSteps) {
    if (currentStep.value === 1 && step1Valid.value) {
      currentStep.value++;
    } else if (
      currentStep.value === 2 &&
      items.value.every((item) => item.product && item.quantity > 0)
    ) {
      currentStep.value++;
    }
  }
};

const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--;
  }
};

const submitInvoice = async() => {
  // Here you would typically send the data to your backend API
  console.log('Submitting invoice:', {
    invoiceDetails: invoiceDetails.value,
    items: items.value,
    total: total.value
  })
  const invoiceData = {
    company: {
      name: invoiceDetails.value.customerName,
      mobileNo: invoiceDetails.value.customerPhone,
      email: invoiceDetails.value.customerEmail,
      gstin: invoiceDetails.value.customerGSTIN,
      address: invoiceDetails.value.customerAddress,
    },
    invoiceToDetails: {
      paymentDate: invoiceDetails.value.invoiceDate,
      name: invoiceDetails.value.recipientName,
      mobileNo: invoiceDetails.value.recipientPhone,
      emailId: invoiceDetails.value.recipientEmail,
      pincode: invoiceDetails.value.recipientPincode,
      city: invoiceDetails.value.recipientCity,
      state: invoiceDetails.value.recipientState,
      address: invoiceDetails.value.recipientAddress,
    },
    transactionDetails: items.value.map(item => ({
      product: item.product,
      rate: item.rate,
      quantity: item.quantity,
      discount: item.discount,
      total: item.total
    })),
    createdBy: "USER"
  }
  const response = await axios.post("https://invoices-codenicely-be.onrender.com/api/invoices/add", invoiceData)
  router.push('/dashboard')
}

const cancelInvoice = () => {
  router.push("/dashboard");
};

const paymentModes = [
  "Cash",
  "Credit Card",
  "Debit Card",
  "UPI",
  "Bank Transfer",
];
const invoiceNumber = "#" + Math.floor(1000000 + Math.random() * 9000000);

const logout = () => {
  localStorage.removeItem("token");
  router.push("/");
};
const addItemButton = () => {
  const newId =
    items.value.length > 0
      ? Math.max(...items.value.map((item) => item.id)) + 1
      : 1;
  items.value.push({
    id: newId,
    product: "",
    rate: 0,
    quantity: 0,
    discount: 0,
    total: 0,
  });
};
onMounted(async () => {
  if (route.params.id) {
    isEditMode.value = true;
    try {
      const response = await axios.get(
        `https://invoices-codenicely-be.onrender.com/api/invoices/${route.params.id}`
      );
      invoiceDetails.value = response.data.invoice.invoiceToDetails;
      console.log(JSON.stringify(response));
    } catch (error) {
      console.error("Error fetching invoice:", error);
    }
  }
});
</script>

<template>
  <div class="invoice-layout">
    <!-- Sidebar -->
    <div class="sidebar">
      <div class="logo-container pa-4">
        <div class="d-flex align-center">
          <v-img
            src="https://raw.githubusercontent.com/devicons/devicon/master/icons/vuejs/vuejs-original.svg"
            max-width="40"
            class="logo"
          />
          <div class="ml-2 text-h6 font-weight-bold text-black">
            CODE<span class="text-green">NICELY</span>
          </div>
        </div>
      </div>

      <v-list>
        <v-list-item title="Team" class="sidebar-item"></v-list-item>
        <v-list-item title="Invoice" class="sidebar-item active"></v-list-item>
      </v-list>

      <div class="logout-container">
        <v-btn
          variant="outlined"
          color="error"
          block
          class="logout-btn"
          @click="logout"
        >
          Logout
        </v-btn>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <!-- Header -->
      <div class="header d-flex align-center px-4 py-2">
        <v-btn icon @click="router.push('/dashboard')">
          <v-icon>mdi-arrow-left</v-icon>
        </v-btn>
        <h1 class="text-h6 ml-2">Add New Invoice</h1>
      </div>

      <!-- Stepper -->
      <div class="stepper-container">
        <div class="stepper-header d-flex justify-center mb-6">
          <div
            class="stepper-item"
            :class="{ active: currentStep === 1, completed: currentStep > 1 }"
          >
            <div class="stepper-circle">1</div>
            <div class="stepper-label">Invoice Details</div>
          </div>
          <div class="stepper-line"></div>
          <div
            class="stepper-item"
            :class="{ active: currentStep === 2, completed: currentStep > 2 }"
          >
            <div class="stepper-circle">2</div>
            <div class="stepper-label">Transaction Details</div>
          </div>

          <div class="stepper-line"></div>
          <div class="stepper-item" :class="{ active: currentStep === 3 }">
            <div class="stepper-circle">3</div>
            <div class="stepper-label">Preview</div>
          </div>
        </div>

        <!-- Step Content -->
        <div
          class="step-content pa-6"
          style="background-color: #ffffff; color: #333"
        >
          <!-- Step 1: Invoice Details -->
          <div v-if="currentStep === 1" class="step-1">
            <v-form v-model="step1Valid">
              <!-- Company Details Section -->
              <div class="section-title text-primary font-weight-bold mb-4">
                Company Details
              </div>

              <div class="mb-4">
                <div class="text-subtitle-2 mb-1">Customer Name</div>
                <v-text-field
                  v-model="invoiceDetails.customerName"
                  placeholder="Enter Name"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  class="mb-3"
                  :rules="nameRules"
                ></v-text-field>
              </div>

              <div class="d-flex gap-4 mb-4">
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">Mobile No.</div>
                  <v-text-field
                    v-model="invoiceDetails.customerPhone"
                    placeholder="Enter Mobile No."
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="phoneRules"
                  ></v-text-field>
                </div>
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">Email</div>
                  <v-text-field
                    v-model="invoiceDetails.customerEmail"
                    placeholder="Enter Email"
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="emailRules"
                  ></v-text-field>
                </div>
              </div>

              <div class="mb-4">
                <div class="text-subtitle-2 mb-1">GSTIN</div>
                <v-text-field
                  v-model="invoiceDetails.customerGSTIN"
                  placeholder="Enter GSTIN"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  class="mb-3"
                  :rules="gstinRules"
                ></v-text-field>
              </div>

              <div class="mb-5">
                <div class="text-subtitle-2 mb-1">Customer Address</div>
                <v-textarea
                  v-model="invoiceDetails.customerAddress"
                  placeholder="Enter Address"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  rows="2"
                  class="mb-3"
                  :rules="addressRules"
                ></v-textarea>
              </div>

              <!-- Invoice To Details Section -->
              <div class="section-title text-primary font-weight-bold mb-4">
                Invoice To Details
              </div>

              <div class="mb-4">
                <div class="text-subtitle-2 mb-1">Payment Date</div>
                <v-text-field
                  v-model="invoiceDetails.paymentDate"
                  type="date"
                  placeholder="Select Payment Date"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  class="mb-3"
                ></v-text-field>
              </div>

              <div class="mb-4">
                <div class="text-subtitle-2 mb-1">Name</div>
                <v-text-field
                  v-model="invoiceDetails.recipientName"
                  placeholder="Enter Name"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  class="mb-3"
                  :rules="nameRules"
                ></v-text-field>
              </div>

              <div class="d-flex gap-4 mb-4">
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">Mobile No.</div>
                  <v-text-field
                    v-model="invoiceDetails.recipientPhone"
                    placeholder="Enter Mobile No."
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="phoneRules"
                  ></v-text-field>
                </div>
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">Email ID</div>
                  <v-text-field
                    v-model="invoiceDetails.recipientEmail"
                    placeholder="Enter Email"
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="emailRules"
                  ></v-text-field>
                </div>
              </div>

              <div class="d-flex gap-4 mb-4">
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">Pincode</div>
                  <v-text-field
                    v-model="invoiceDetails.recipientPincode"
                    placeholder="Enter Pincode"
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="pincodeRules"
                  ></v-text-field>
                </div>
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">City</div>
                  <v-text-field
                    v-model="invoiceDetails.recipientCity"
                    placeholder="Enter City"
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="cityRules"
                  ></v-text-field>
                </div>
                <div class="flex-1">
                  <div class="text-subtitle-2 mb-1">State</div>
                  <v-text-field
                    v-model="invoiceDetails.recipientState"
                    placeholder="Enter State"
                    variant="outlined"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-3"
                    :rules="stateRules"
                  ></v-text-field>
                </div>
              </div>

              <div class="mb-5">
                <div class="text-subtitle-2 mb-1">Address</div>
                <v-textarea
                  v-model="invoiceDetails.recipientAddress"
                  placeholder="Enter Address"
                  variant="outlined"
                  density="comfortable"
                  hide-details="auto"
                  rows="2"
                  class="mb-3"
                  :rules="addressRules"
                ></v-textarea>
              </div>
            </v-form>

            <div class="d-flex justify-end mt-6">
              <v-btn
                color="primary"
                rounded
                @click="nextStep"
                :disabled="!step1Valid"
              >
                Next
              </v-btn>
            </div>
          </div>

          <!-- Step 2: Transaction Details -->
          <div v-if="currentStep === 2" class="step-2">
            <div class="section-title text-primary font-weight-bold mb-4">
              Transaction Details
              <button @click="addItemButton" class="btn btn-primary">+</button>

            </div>


            <v-form v-model="step2Valid">
              <div class="transaction-table">
                <div class="transaction-header">
                  <div class="product-col">Product</div>
                  <div class="rate-col">Rate</div>
                  <div class="quantity-col">Quantity</div>
                  <div class="discount-col">Discount (in Rs.)</div>
                  <div class="total-col">Total</div>
                  <div class="action-col"></div>
                </div>

                <div
                  v-for="item in items"
                  :key="item.id"
                  class="transaction-row"
                >
                  <div class="product-col">
                    <v-text-field
                      v-model="item.product"
                      placeholder="Enter Product"
                      variant="outlined"
                      density="compact"
                      hide-details
                      :rules="productRules"
                    ></v-text-field>
                  </div>
                  <div class="rate-col">
                    <v-text-field
                      v-model.number="item.rate"
                      placeholder="Enter Rate"
                      variant="outlined"
                      density="compact"
                      hide-details
                      type="number"
                      :rules="rateRules"
                      @input="updateItemTotal(item)"
                    ></v-text-field>
                  </div>
                  <div class="quantity-col">
                    <v-text-field
                      v-model.number="item.quantity"
                      placeholder="Enter Quantity"
                      variant="outlined"
                      density="compact"
                      hide-details
                      type="number"
                      :rules="quantityRules"
                      @input="updateItemTotal(item)"
                    ></v-text-field>
                  </div>
                  <div class="discount-col">
                    <v-text-field
                      v-model.number="item.discount"
                      placeholder="Enter Discount"
                      variant="outlined"
                      density="compact"
                      hide-details
                      type="number"
                      @input="updateItemTotal(item)"
                    ></v-text-field>
                  </div>
                  <div class="total-col">{{ item.total }}</div>
                  <div class="action-col">
                    <v-btn
                      icon
                      color="error"
                      size="small"
                      @click="removeItem(item.id)"
                      :disabled="items.length === 1"
                    >
                      <v-icon>mdi-close</v-icon>
                    </v-btn>
                  </div>
                </div>
              </div>

              <div class="d-flex justify-end mt-4">
                <div class="total-summary">
                  <div class="d-flex justify-space-between">
                    <div>Total</div>
                    <div>Rs. {{ total }}</div>
                  </div>
                </div>
              </div>
            </v-form>

            <div class="d-flex justify-end gap-4 mt-6">
              <v-btn
                color="secondary"
                variant="outlined"
                rounded
                @click="prevStep"
              >
                Back
              </v-btn>
              <v-btn
                color="primary"
                rounded
                @click="nextStep"
                :disabled="!step2Valid"
              >
                Next
              </v-btn>
            </div>
          </div>

          <!-- Step 3: Preview -->
          <div v-if="currentStep === 3" class="step-3">
            <div class="invoice-preview">
              <div class="d-flex justify-space-between mb-4">
                <div>
                  <h2 class="text-h5 font-weight-bold">Company Name</h2>
                  <h1 class="text-h4 text-green font-weight-bold mt-4">
                    Invoice
                  </h1>
                </div>
                <div class="text-right">
                  <div>Mobile: +91-{{ invoiceDetails.customerPhone }}</div>
                  <div>Email: {{ invoiceDetails.customerEmail }}</div>
                  <div>GSTIN: {{ invoiceDetails.customerGSTIN }}</div>
                  <div class="text-h6 mt-4">{{ invoiceNumber }}</div>
                  <div class="mt-4">
                    Date:
                    {{
                      new Date()
                        .toLocaleDateString("en-GB")
                        .split("/")
                        .join("-")
                    }}
                  </div>
                </div>
              </div>

              <div class="invoice-to mt-8 mb-6">
                <h3 class="text-subtitle-1 font-weight-bold">INVOICE TO:</h3>
                <div class="mt-2">Name: {{ invoiceDetails.recipientName }}</div>
                <div>Mobile: +91-{{ invoiceDetails.recipientPhone }}</div>
                <div>Email: {{ invoiceDetails.recipientEmail }}</div>
                <div>
                  Address: {{ invoiceDetails.recipientAddress }},
                  {{ invoiceDetails.recipientCity }},
                  {{ invoiceDetails.recipientState }}
                  {{ invoiceDetails.recipientPincode }}
                </div>
              </div>

              <table class="invoice-table">
                <thead>
                  <tr>
                    <th>Product</th>
                    <th>Rate</th>
                    <th>Quantity</th>
                    <th>Discount</th>
                    <th>Total</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in items" :key="item.id">
                    <td>{{ item.product }}</td>
                    <td>{{ item.rate }}</td>
                    <td>{{ item.quantity }}</td>
                    <td>{{ item.discount }}</td>
                    <td>{{ item.total }}</td>
                  </tr>
                </tbody>
              </table>

              <div class="d-flex justify-end mt-4">
                <div class="total-summary">
                  <div class="d-flex justify-space-between">
                    <div>Total</div>
                    <div>Rs. {{ total }}</div>
                  </div>
                </div>
              </div>
            </div>

            <div class="d-flex justify-end gap-4 mt-6">
              <v-btn
                color="secondary"
                variant="outlined"
                rounded
                @click="prevStep"
              >
                Back
              </v-btn>
              <v-btn color="primary" rounded @click="submitInvoice">
                Done
              </v-btn>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.invoice-layout {
  display: flex;
  height: 100vh;
}

.sidebar {
  width: 250px;
  background-color: white;
  border-right: 1px solid #e0e0e0;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.sidebar-item {
  border-radius: 0;
}

.sidebar-item.active {
  background-color: #1976d2;
  color: white;
}

.logout-container {
  margin-top: auto;
  padding: 16px;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;
}

.header {
  background-color: white;
  border-bottom: 1px solid #e0e0e0;
}

.stepper-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.stepper-header {
  display: flex;
  align-items: center;
  justify-content: center;
}

.stepper-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.stepper-circle {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: #e0e0e0;
  color: #757575;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

.stepper-item.active .stepper-circle {
  background-color: #1976d2;
  color: white;
}

.stepper-item.completed .stepper-circle {
  background-color: #4caf50;
  color: white;
}

.stepper-label {
  margin-top: 8px;
  font-size: 12px;
  color: #757575;
}

.stepper-item.active .stepper-label {
  color: #1976d2;
  font-weight: bold;
}

.stepper-line {
  flex: 1;
  height: 2px;
  background-color: #e0e0e0;
  margin: 0 10px;
  max-width: 100px;
}

.step-content {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.btn-primary {
  background-color: #1976d2;
  color: white;
  border: none;
  padding: 8px 16px;
  font-size: 14px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-primary:hover {
  background-color: #155a9e;
}

.btn-primary:disabled {
  background-color: #a1c5e5;
  cursor: not-allowed;
}

.section-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 16px;
  color: #1976d2;
}

.flex-1 {
  flex: 1;
}

.gap-4 {
  gap: 16px;
}

/* Transaction Table Styles */
.transaction-table {
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.transaction-header {
  display: flex;
  background-color: #f5f5f5;
  padding: 12px 8px;
  font-weight: bold;
}

.transaction-row {
  display: flex;
  padding: 8px;
  border-top: 1px solid #e0e0e0;
}

.product-col {
  flex: 3;
  padding: 0 4px;
}

.rate-col,
.quantity-col,
.discount-col,
.total-col {
  flex: 1;
  padding: 0 4px;
  display: flex;
  align-items: center;
}

.action-col {
  width: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.total-summary {
  width: 200px;
  padding: 8px 16px;
  background-color: #f5f5f5;
  border-radius: 4px;
  font-weight: bold;
}

/* Invoice Preview Styles */
.invoice-preview {
  padding: 20px;
}

.invoice-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.invoice-table th {
  background-color: #f9f9f9;
  padding: 10px;
  text-align: left;
  border: 1px solid #e0e0e0;
}

.invoice-table td {
  padding: 10px;
  border: 1px solid #e0e0e0;
}

.text-green {
  color: #4caf50;
}
</style>
