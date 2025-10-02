<script setup>
import Datee from "@/components/Datee.vue";
import Select from "@/components/Select.vue";
import { useAuthStore } from "@/store/auth";
import { useUrl } from "@/store/url";
import { onBeforeMount, onMounted } from "vue";
import { ref, watch } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import CloseIcon from "@/components/CloseIcon.vue";

const auth = useAuthStore();
const url = useUrl();
const router = useRouter();

const current = ref("rr");

const dateOptions = ["Today", "Yesterday", "Last 7 days", "Last 30 Days"];
const today = new Date().toISOString().split("T")[0];
const startt = ref(today);
const endd = ref(today);
const rr = ref(null);
const rrGrouped = ref(null);

const crStart = ref(today);
const crEnd = ref(today);
const cr = ref(null);

const shops = ref([]);
const currentShop = ref(null);

const ms = ref(null);
const noOfCashiers = ref(3);
const shopLimit = ref(10000);
const shopName = ref(null);
const serverType = ref("convex");
const error = ref(false);
const errorMessage = ref(null);
const success = ref(false);
const successMessage = ref(null);

const editShopName = ref(null);
const editServerName = ref(null);
const editShopLimit = ref(null);
const neditShopName = ref(null);
const neditServerName = ref(null);
const neditShopLimit = ref(null);

const cashierShopName = ref(null);
const cashierServerName = ref(null);
const cashierShopLimit = ref(null);
const ncashierShopName = ref(null);
const ncashierServerName = ref(null);
const ncashierShopLimit = ref(null);
const cashierName = ref(null);

const cashiers = ref([]);

const createNewShopModal = ref(false);
const editShopModal = ref(false);
const editShopModal2 = ref(false);
const createNewCashierModal = ref(false);

async function getData() {
  const username = auth?.user?.username;
  const type = auth?.user?.type;
  console.log(startt.value);
  const res = await axios.post(`${url.url}/api/getData`, {
    username,
    feedId: type,
    startt: startt.value,
    endd: endd.value,
  });

  console.log(res.data);
  rr.value = res.data.result;
  rrGrouped.value = res.data.groupRes;
}

async function getCashierData() {
  const username = auth?.user?.username;
  const type = auth?.user?.type;
  console.log(startt.value);
  const res = await axios.post(`${url.url}/api/getCashierData`, {
    username,
    feedId: type,
    startt: crStart.value,
    endd: crEnd.value,
  });

  console.log(res.data);
  cr.value = res.data.groupRes;
  // rr.value = res.data.result;
  // rrGrouped.value = res.data.groupRes;
}

async function getShopList() {
  ms.value = null;
  const username = auth?.user?.username;
  const type = auth?.user?.type;
  console.log(startt.value);
  const res = await axios.post(`${url.url}/api/getShopList`, {
    username,
    feedId: type,
  });

  console.log(res.data.cashierCount);
  ms.value = res.data.cashierCount;
  shops.value = ms.value.map((item) => item.user);
  console.log(shops.value);
  currentShop.value = shops.value[0];
}

async function createNewSHop(params) {
  error.value = false;
  errorMessage.value = null;
  if (shopName.value === null) {
    error.value = true;
    errorMessage.value = "Shop name is required.";
    return;
  } else if (noOfCashiers.value === 0) {
    error.value = true;
    errorMessage.value = "Minimum number of cashiers is 1.";
    return;
  } else if (shopLimit.value < 5000 && shopLimit.value > 50000) {
    error.value = true;
    errorMessage.value = "Limit must be between 5000 and 50000.";
    return;
  }

  const res = await axios.post(`${url.url}/api/createNewShop`, {
    shopName: shopName.value,
    noOfCashiers: noOfCashiers.value,
    shopLimit: shopLimit.value,
    serverType: serverType.value,
    user: auth?.user?.username,
  });

  if (!res.data.success) {
    error.value = true;
    errorMessage.value = res.data.message;
  }
  if (res.data.success) {
    success.value = true;
    successMessage.value = res.data.message;
  }
}

async function deactivateShop(shop, b) {
  const res = await axios.post(`${url.url}/api/closeShop`, {
    user: auth?.user?.username,
    shop,
    b,
  });
  if (res.data.success) {
    getShopList();
  }
}

async function deactivateCashier(shop, b, c) {
  const res = await axios.post(`${url.url}/api/closeCashier`, {
    user: auth?.user?.username,
    shop,
    b,
    c,
  });
  if (res.data.success) {
    getCashiersList();
  }
}

async function updateShop(params) {
  error.value = false;
  errorMessage.value = null;
  success.value = false;
  successMessage.value = null;
  if (editShopName.value === null) {
    error.value = true;
    errorMessage.value = "Shop name is required.";
    return;
  } else if (editShopLimit.value < 5000 && editShopLimit.value > 50000) {
    error.value = true;
    errorMessage.value = "Limit must be between 5000 and 50000.";
    return;
  }

  const res = await axios.post(`${url.url}/api/updateShop`, {
    shopName: editShopName.value,
    shopLimit: editShopLimit.value,
    serverType: editServerName.value,
    nshopName: neditShopName.value,
    nshopLimit: neditShopLimit.value,
    nserverType: neditServerName.value,
    user: auth?.user?.username,
  });

  if (!res.data.success) {
    error.value = true;
    errorMessage.value = res.data.message;
  }
  if (res.data.success) {
    success.value = true;
    successMessage.value = res.data.message;
  }
  getShopList();
}

async function updateCashier(params) {
  error.value = false;
  errorMessage.value = null;
  success.value = false;
  successMessage.value = null;
  if (cashierShopName.value === null) {
    error.value = true;
    errorMessage.value = "Shop name is required.";
    return;
  } else if (cashierShopLimit.value < 5000 && cashierShopLimit.value > 50000) {
    error.value = true;
    errorMessage.value = "Limit must be between 5000 and 50000.";
    return;
  }

  const res = await axios.post(`${url.url}/api/updateCashier`, {
    shopName: cashierShopName.value,
    shopLimit: cashierShopLimit.value,
    serverType: cashierServerName.value,
    nshopName: ncashierShopName.value,
    nshopLimit: ncashierShopLimit.value,
    nserverType: ncashierServerName.value,
    user: auth?.user?.username,
    cashierName: cashierName.value,
  });

  if (!res.data.success) {
    error.value = true;
    errorMessage.value = res.data.message;
  }
  if (res.data.success) {
    success.value = true;
    successMessage.value = res.data.message;
  }
  getCashiersList();
}

async function getCashiersList(params) {
  setTimeout(async () => {
    const res = await axios.post(`${url.url}/api/getCashierList`, {
      user: auth?.user?.username,
      shop: currentShop.value,
    });
    if (res.data.success) {
      cashiers.value = res.data.cashiers;
      console.log(cashiers.value);
    }
  }, 500);
}

const unclaimed = ref([]);
const det = ref(null);

async function search(a, b) {
  const res = await axios.post(`${url.url}/api/searchUnclaimed`, {
    user: auth?.user?.username,
    shopname: a,
    cashiername: b,
  });

  unclaimed.value = res.data.unclaimed;
  det.value = res.data.shopnamee + "." + res.data.cashiernamee;
}

watch(currentShop, (newVal, oldVal) => {
  console.log("changed to:", newVal);
  getCashiersList();
});

function handleLogout() {
  auth.logout();
}

onMounted(async () => {
  const ok = await auth.checkSession();
  if (!ok) {
    router.push("/Login");
  }
  getData();
  // getShopList();
});
</script>

<template>
  <div
    class="w-full h-[6vh] md:h-[8vh] bg-[#f6f6f6] shadow-md shadow-[#999] flex justify-between items-center px-4"
  >
    <div class="font-bold text-xs md:text-xl tracking-wider">Retail Report</div>
    <div class="flex items-center gap-4">
      <div class="text-[#37b34a] text-xs md:text-lg">
        Hi, {{ auth?.user?.username }}
      </div>
      <div
        class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300"
      >
        Change password
      </div>
      <div
        @click="handleLogout"
        class="bg-[#3b6ad7] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#1455ed] cursor-pointer transition-colors duration-300"
      >
        Logout
      </div>
    </div>
  </div>

  <div class="flex flex-col md:flex-row gap-1 w-full h-[94vh] md:h-[92vh]">
    <div
      class="w-full bg-[#f6f6f6] mt-0.5 md:w-[25%] md:h-full flex flex-row md:flex-col gap-4 justify-center md:justify-start md:items-center py-2 md:py-4 text-xs md:text-lg"
    >
      <div
        class="px-2 rounded-lg py-1 cursor-pointer tracking-wider text-center hover:opacity-50 transition-all duration-300"
        :class="`${current === 'rr' ? 'bg-[#8c8c8c] text-white' : 'bg-white'}`"
        @click="
          current = 'rr';
          getData();
        "
      >
        Retail Report
      </div>
      <div
        class="px-2 rounded-lg py-1 cursor-pointer tracking-wider text-center hover:opacity-50 transition-all duration-300"
        :class="`${current === 'cr' ? 'bg-[#8c8c8c] text-white' : 'bg-white'}`"
        @click="
          current = 'cr';
          getCashierData();
        "
      >
        Cashier Report
      </div>
      <div
        class="px-2 rounded-lg py-1 cursor-pointer tracking-wider text-center hover:opacity-50 transition-all duration-300"
        :class="`${current === 'ms' ? 'bg-[#8c8c8c] text-white' : 'bg-white'}`"
        @click="
          current = 'ms';
          getShopList();
        "
      >
        Manage Shop
      </div>
      <div
        class="px-2 rounded-lg py-1 cursor-pointer tracking-wider text-center hover:opacity-50 transition-all duration-300"
        :class="`${current === 'mc' ? 'bg-[#8c8c8c] text-white' : 'bg-white'}`"
        @click="
          current = 'mc';
          getShopList();
          getCashiersList();
        "
      >
        Manage Cashier
      </div>
    </div>
    <div
      class="w-full h-full bg-[#dedede] mt-0.5 md:w-[75%] md:h-full flex flex-row md:flex-col gap-4 justify-center md:justify-start md:items-center py-2 md:py-4 px-2"
    >
      <div v-if="current === 'rr'" class="p-2 w-full">
        <div class="text-lg">Retail Report</div>
        <hr class="w-full border-[1px] border-[#666]" />
        <!-- Filter  -->
        <div
          class="w-full rounded-lg bg-white flex flex-col flex-wrap md:flex-row mt-2 p-2 md:py-4 gap-2 md:items-center text-xs md:text-lg"
        >
          <div class="flex-1 hidden">
            Date
            <Select :options="dateOptions" />
          </div>
          <div class="flex-1">
            From
            <Datee v-model="startt" />
          </div>
          <div class="flex-1">
            To
            <Datee v-model="endd" />
          </div>
          <div class="flex-1">
            Shop
            <Select :options="dateOptions" />
          </div>
          <div class="flex-1">
            <div
              @click="getData()"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 text-center"
            >
              Load Report
            </div>
          </div>
        </div>

        <!-- Data  -->
        <div
          class="w-full rounded-lg bg-white flex flex-col flex-wrap md:flex-row mt-2 p-2 md:py-4 gap-2 md:items-center"
        >
          <div
            class="flex-1 bg-[#F97316] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Net Balance</span>
            <span
              >Br.
              {{
                (
                  (rr?.totalStake ?? 0).toFixed(2) -
                  (rr?.redeemedValue ?? 0).toFixed(2) -
                  (rr?.cancelledStake ?? 0).toFixed(2)
                ).toFixed(2)
              }}</span
            >
          </div>
          <div
            class="flex-1 bg-[#0891B2] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Tickets</span>
            <span>{{ rr?.totalTicket }} tickets</span>
          </div>
          <div
            class="flex-1 bg-[#15803D] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Gross Stake</span>
            <span>Br. {{ (rr?.totalStake ?? 0).toFixed(2) }}</span>
          </div>
          <div
            class="flex-1 bg-[#1E293B] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Claimed Winnings</span>
            <span>Br. {{ (rr?.redeemedValue ?? 0).toFixed(2) }}</span>
          </div>
        </div>

        <!-- table data  -->
        <table
          class="w-full text-xs md:text-base my-4 md:my-2 bg-white rounded-lg p-2"
        >
          <thead>
            <tr class="font-semibold bg-[#efefef] py-2">
              <th class="text-center">No</th>
              <th class="text-center">Shop Name</th>
              <th class="text-center">Total Stake</th>
              <th class="text-center">Winnings</th>
              <th class="text-center">Cancelled</th>
              <th class="text-center">Net Balance</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(g, i) in rrGrouped" :key="g.username">
              <td class="text-center">{{ i + 1 }}</td>
              <td class="text-center">{{ g?.username }}</td>
              <td class="text-center">{{ (g?.sumStake ?? 0).toFixed(2) }}</td>
              <td class="text-center">{{ (g?.sumWin ?? 0).toFixed(2) }}</td>
              <td class="text-center">
                {{ (g?.sumCancelled ?? 0).toFixed(2) }}
              </td>
              <td class="text-center">
                {{
                  (
                    (g?.sumStake ?? 0) -
                    (g?.sumWin ?? 0) -
                    (g?.sumCancelled ?? 0)
                  ).toFixed(2)
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-if="current === 'cr'" class="p-2 w-full">
        <div class="text-lg">Cashier Report</div>
        <hr class="w-full border-[1px] border-[#666]" />
        <!-- Filter  -->
        <div
          class="w-full rounded-lg bg-white flex flex-col flex-wrap md:flex-row mt-2 p-2 md:py-4 gap-2 md:items-center text-xs md:text-lg"
        >
          <div class="flex-1 hidden">
            Date
            <Select :options="dateOptions" />
          </div>
          <div class="flex-1">
            From
            <Datee v-model="crStart" />
          </div>
          <div class="flex-1">
            To
            <Datee v-model="crEnd" />
          </div>
          <div class="flex-1 hidden">
            Shop
            <Select :options="dateOptions" />
          </div>
          <div class="flex-1">
            <span class="text-white">a</span>
            <div
              @click="getCashierData()"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1.5 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 text-center"
            >
              Load Report
            </div>
          </div>
        </div>

        <!-- Data  -->
        <div
          v-if="false"
          class="w-full rounded-lg bg-white flex flex-col flex-wrap md:flex-row mt-2 p-2 md:py-4 gap-2 md:items-center"
        >
          <div
            class="flex-1 bg-[#F97316] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Net Balance</span>
            <span
              >Br.
              {{
                (
                  (rr?.totalStake ?? 0).toFixed(2) -
                  (rr?.redeemedValue ?? 0).toFixed(2) -
                  (rr?.cancelledStake ?? 0).toFixed(2)
                ).toFixed(2)
              }}</span
            >
          </div>
          <div
            class="flex-1 bg-[#0891B2] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Tickets</span>
            <span>{{ rr?.totalTicket }} tickets</span>
          </div>
          <div
            class="flex-1 bg-[#15803D] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Gross Stake</span>
            <span>Br. {{ (rr?.totalStake ?? 0).toFixed(2) }}</span>
          </div>
          <div
            class="flex-1 bg-[#1E293B] text-white flex flex-col rounded-lg px-4 py-2 text-xs md:text-lg items-center"
          >
            <span>Claimed Winnings</span>
            <span>Br. {{ (rr?.redeemedValue ?? 0).toFixed(2) }}</span>
          </div>
        </div>

        <!-- table data  -->
        <table
          class="w-full text-xs md:text-base my-4 md:my-2 bg-white rounded-lg p-2"
        >
          <thead>
            <tr class="font-semibold bg-[#efefef] py-2">
              <th class="text-center">No</th>
              <th class="text-center">Cashier Name</th>
              <th class="text-center">Total Stake</th>
              <th class="text-center">Winnings</th>
              <th class="text-center">Cancelled</th>
              <th class="text-center">Net Balance</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(g, i) in cr" :key="g.username">
              <td class="text-center">{{ i + 1 }}</td>
              <td class="text-center">{{ g?.username + "." + g?.cashier }}</td>
              <td class="text-center">{{ (g?.sumStake ?? 0).toFixed(2) }}</td>
              <td class="text-center">{{ (g?.sumWin ?? 0).toFixed(2) }}</td>
              <td class="text-center">
                {{ (g?.sumCancelled ?? 0).toFixed(2) }}
              </td>
              <td class="text-center">
                {{
                  (
                    (g?.sumStake ?? 0) -
                    (g?.sumWin ?? 0) -
                    (g?.sumCancelled ?? 0)
                  ).toFixed(2)
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-if="current === 'ms'" class="p-2 w-full">
        <div class="flex justify-between items-center mb-2">
          <div class="text-xs md:text-lg">Manage Shops</div>
          <div
            @click="createNewShopModal = true"
            class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300"
          >
            Create new shop
          </div>
        </div>
        <hr class="w-full border-[1px] border-[#666]" />
        <div class="text-xs md:text-lg mt-4">Shops</div>
        <table
          class="w-full text-xs md:text-base my-4 md:my-2 bg-white rounded-lg p-2"
        >
          <thead>
            <tr class="font-semibold bg-[#efefef] py-2 text-xs md:text-base">
              <th class="text-center">No</th>
              <th class="text-center">Shop Name</th>
              <th class="text-center">No of cashiers</th>
              <th class="text-center">Limit</th>
              <th class="text-center">Owner</th>
              <th class="text-center">Status</th>
              <th class="text-center">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(g, i) in ms" :key="i" class="text-xs md:text-base">
              <td class="text-center">{{ i + 1 }}</td>
              <td class="text-center">{{ g.user }}</td>
              <td class="text-center">{{ g.cashierCount }}</td>
              <td class="text-center">{{ g.balanceLimit }}</td>
              <td class="text-center">{{ auth?.user?.username }}</td>
              <td
                class="text-center"
                :class="`${
                  g.isBlocked === 1 ? 'text-red-500' : 'text-green-500'
                }`"
              >
                {{ g.isBlocked == 1 ? "Inactive" : "Active" }}
              </td>
              <td
                class="text-center flex gap-1 justify-center text-xs md:text-base"
              >
                <div
                  @click="deactivateShop(g.user, g.isBlocked)"
                  class="text-xs md:text-base px-2 py-1 rounded text-white cursor-pointer transition-colors duration-300 w-[40%]"
                  :class="`${
                    g.isBlocked === 1
                      ? 'bg-green-500 hover:bg-green-400 '
                      : 'bg-red-500 hover:bg-red-400'
                  }`"
                >
                  {{ g.isBlocked == 1 ? "Activate" : "Deactivate" }}
                </div>
                <div
                  @click="
                    editShopName = g.user;
                    editShopLimit = g.balanceLimit;
                    editServerName = g.source === 1 ? 'convex' : 'fiveking';
                    neditShopName = g.user;
                    neditShopLimit = g.balanceLimit;
                    neditServerName = g.source === 1 ? 'convex' : 'fiveking';
                    editShopModal = true;
                  "
                  class="bg-yellow-500 text-xs md:text-base px-2 py-1 rounded text-white hover:bg-yellow-400 cursor-pointer transition-colors duration-300 w-[40%]"
                >
                  Edit
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-if="current === 'mc'" class="p-2 w-full">
        <div class="flex justify-between items-center mb-2">
          <div class="text-xs md:text-lg">Manage Cashiers</div>
          <div class="flex items-center gap-1 w-[40%]">
            <select
              v-model="currentShop"
              class="border p-[5.4px] rounded w-full"
            >
              <option v-for="opt in shops" :key="opt" :value="opt">
                {{ opt }}
              </option>
            </select>
            <div
              @click="createNewCashierModal = true"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 w-[40%]"
            >
              Create new cashier
            </div>
          </div>
        </div>
        <hr class="w-full border-[1px] border-[#666]" />
        <div class="text-xs md:text-lg mt-4">Cashiers</div>
        <table
          class="w-full text-xs md:text-base my-4 md:my-2 bg-white rounded-lg p-2"
        >
          <thead>
            <tr class="font-semibold bg-[#efefef] py-2 text-xs md:text-base">
              <th class="text-center">No</th>
              <th class="text-center">Shop Name</th>
              <th class="text-center">User Name</th>
              <th class="text-center">Limit</th>
              <th class="text-center">Status</th>
              <th class="text-center">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(g, i) in cashiers"
              :key="i"
              class="text-xs md:text-base"
            >
              <td class="text-center">{{ i + 1 }}</td>
              <td class="text-center">{{ g.user }}</td>
              <td class="text-center">{{ g.user + "." + g.cashier }}</td>
              <td class="text-center">{{ g.balancelimit }}</td>
              <td
                class="text-center"
                :class="`${
                  g.isBlocked === 1 ? 'text-red-500' : 'text-green-500'
                }`"
              >
                {{ g.isBlocked == 1 ? "Inactive" : "Active" }}
              </td>
              <td
                class="text-center flex gap-1 justify-center text-xs md:text-base"
              >
                <div
                  @click="deactivateCashier(g.user, g.isBlocked, g.cashier)"
                  class="text-xs md:text-base px-2 py-1 rounded text-white cursor-pointer transition-colors duration-300 w-[40%]"
                  :class="`${
                    g.isBlocked === 1
                      ? 'bg-green-500 hover:bg-green-400 '
                      : 'bg-red-500 hover:bg-red-400'
                  }`"
                >
                  {{ g.isBlocked == 1 ? "Activate" : "Deactivate" }}
                </div>
                <div
                  @click="
                    cashierShopName = g.cashier;
                    cashierShopLimit = g.balancelimit;
                    cashierServerName = g.source === 1 ? 'convex' : 'fiveking';
                    (cashierName = g.user), (ncashierShopName = g.cashier);
                    ncashierShopLimit = g.balancelimit;
                    ncashierServerName = g.source === 1 ? 'convex' : 'fiveking';
                    editShopModal2 = true;
                  "
                  class="bg-yellow-500 text-xs md:text-base px-2 py-1 rounded text-white hover:bg-yellow-400 cursor-pointer transition-colors duration-300 w-[40%]"
                >
                  Edit
                </div>
                <div
                  @click="search(g.user, g.cashier)"
                  class="text-xs md:text-base px-2 py-1 rounded text-white cursor-pointer transition-colors duration-300 w-[40%] bg-blue-500 hover:bg-blue-400"
                >
                  Search
                </div>
              </td>
            </tr>
          </tbody>
        </table>

        <div v-if="unclaimed.length > 0" class="mt-2 bg-white rounded-lg p-2">
          Unclaimed Tickets for Today for
          <span class="text-green-500">{{ det }}</span>
          <div v-for="u in unclaimed">
            {{ u.ticketId }}
          </div>
        </div>
      </div>

      <div
        v-if="createNewShopModal"
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-20 p-2"
      >
        <div class="bg-white p-6 rounded-lg shadow-lg w-full md:w-1/2 relative">
          <CloseIcon
            class="absolute top-2 right-2"
            @click="createNewShopModal = false"
          />
          <h2 class="text-lg font-semibold mb-4">Create new shop</h2>
          <p>Password is <span class="text-yellow-500">betman1234</span></p>
          <div class="flex flex-col gap-2 w-full">
            <div class="flex flex-col gap-1">
              Shop Name
              <input
                type="text"
                v-model="shopName"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div class="flex flex-col gap-1">
              Server Config
              <Select v-model="serverType" :options="['convex', 'fiveking']" />
            </div>

            <div class="flex flex-col gap-1">
              Number of Cashiers
              <input
                type="number"
                v-model="noOfCashiers"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div class="flex flex-col gap-1">
              Shop Cashier Limit
              <input
                type="number"
                v-model="shopLimit"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div
              @click="createNewSHop()"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 text-center"
            >
              Create Shop
            </div>
            <div v-if="error" class="text-red-500 text-center">
              {{ errorMessage }}
            </div>
            <div v-if="success" class="text-green-500 text-center">
              {{ successMessage }}
            </div>
          </div>
        </div>
      </div>

      <div
        v-if="editShopModal"
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-20 p-2"
      >
        <div class="bg-white p-6 rounded-lg shadow-lg w-full md:w-1/2 relative">
          <CloseIcon
            class="absolute top-2 right-2"
            @click="editShopModal = false"
          />
          <h2 class="text-lg font-semibold mb-4">Edit shop</h2>
          <div class="flex flex-col gap-2 w-full">
            <div class="flex flex-col gap-1">
              Shop Name
              <input
                type="text"
                v-model="editShopName"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div class="flex flex-col gap-1">
              Server Config
              <Select
                v-model="editServerName"
                :options="['convex', 'fiveking']"
              />
            </div>

            <!-- <div class="flex flex-col gap-1">
              Number of Cashiers
              <input
                type="number"
                v-model="noOfCashiers"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div> -->

            <div class="flex flex-col gap-1">
              Shop Cashier Limit
              <input
                type="number"
                v-model="editShopLimit"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div
              @click="updateShop()"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 text-center"
            >
              Update Shop
            </div>
            <div v-if="error" class="text-red-500 text-center">
              {{ errorMessage }}
            </div>
            <div v-if="success" class="text-green-500 text-center">
              {{ successMessage }}
            </div>
          </div>
        </div>
      </div>

      <div
        v-if="editShopModal2"
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-20 p-2"
      >
        <div class="bg-white p-6 rounded-lg shadow-lg w-full md:w-1/2 relative">
          <CloseIcon
            class="absolute top-2 right-2"
            @click="editShopModal2 = false"
          />
          <h2 class="text-lg font-semibold mb-4">Edit cashier</h2>
          <div class="flex flex-col gap-2 w-full">
            <div class="flex flex-col gap-1">
              Cashier Name
              <input
                type="text"
                v-model="cashierShopName"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
                disabled
              />
            </div>

            <div class="flex flex-col gap-1">
              Server Config
              <Select
                v-model="cashierServerName"
                :options="['convex', 'fiveking']"
              />
            </div>

            <!-- <div class="flex flex-col gap-1">
              Number of Cashiers
              <input
                type="number"
                v-model="noOfCashiers"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div> -->

            <div class="flex flex-col gap-1">
              Shop Cashier Limit
              <input
                type="number"
                v-model="cashierShopLimit"
                class="p-2 bg-[#efefef] rounded outline-none border border-[#37b34a]"
              />
            </div>

            <div
              @click="updateCashier()"
              class="bg-[#37b34a] text-xs md:text-base px-2 py-1 rounded text-white hover:bg-[#21d64e] cursor-pointer transition-colors duration-300 text-center"
            >
              Update Shop
            </div>
            <div v-if="error" class="text-red-500 text-center">
              {{ errorMessage }}
            </div>
            <div v-if="success" class="text-green-500 text-center">
              {{ successMessage }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
