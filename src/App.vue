<script setup lang="ts">
import { ref, computed, watch } from "vue";

interface Transaction {
  id: number;
  desc: string;
  amount: number;
  type: "income" | "expense";
}

const transactions = ref<Transaction[]>([]);

const description = ref("");
const amount = ref<number | null>(null);
const type = ref<"income" | "expense">("income");

const filterType = ref("all");
const budgetLimit = ref(1000);

const notificationLog = ref<string[]>([]);

function addTransaction() {
  if (!description.value || !amount.value) return;

  transactions.value.push({
    id: Date.now(),
    desc: description.value,
    amount: amount.value,
    type: type.value,
  });

  description.value = "";
  amount.value = null;
  type.value = "income";
}

function deleteTransaction(id: number) {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id,
  );
}

function clearAll() {
  transactions.value = [];
}

/* Computed Properties */

const filteredTransactions = computed(() => {
  if (filterType.value === "all") {
    return transactions.value;
  }

  return transactions.value.filter(
    (transaction) => transaction.type === filterType.value,
  );
});

const totalIncome = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.type === "income")
    .reduce((sum, transaction) => sum + transaction.amount, 0);
});

const totalExpenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.type === "expense")
    .reduce((sum, transaction) => sum + transaction.amount, 0);
});

const balance = computed(() => {
  return totalIncome.value - totalExpenses.value;
});

const isOverBudget = computed(() => {
  return totalExpenses.value > budgetLimit.value;
});

const expensePercentage = computed(() => {
  return Math.min((totalExpenses.value / budgetLimit.value) * 100, 100);
});

const categorySummary = computed(() => {
  return transactions.value.reduce(
    (acc, transaction) => {
      acc[transaction.type] = (acc[transaction.type] || 0) + transaction.amount;

      return acc;
    },
    {} as Record<string, number>,
  );
});

const budgetStatus = computed(() => {
  if (isOverBudget.value) {
    return "Over Budget";
  }

  if (expensePercentage.value >= 80) {
    return "Near Budget Limit";
  }

  return "Healthy";
});

/* Watchers */

watch(
  transactions,
  (newTransactions) => {
    localStorage.setItem("transactions", JSON.stringify(newTransactions));
  },
  { deep: true },
);

watch(balance, (newBalance) => {
  if (newBalance < 0) {
    alert("Warning: Balance is below $0");
  }
});

watch(isOverBudget, (value) => {
  if (value) {
    notificationLog.value.push(
      `Budget exceeded at ${new Date().toLocaleString()}`,
    );
  }
});

/* Load Local Storage */

const savedTransactions = localStorage.getItem("transactions");

if (savedTransactions) {
  transactions.value = JSON.parse(savedTransactions);
}
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-8">
    <div class="max-w-5xl mx-auto">
      <h1 class="text-4xl font-bold text-center mb-8">
        💰 Personal Finance Tracker
      </h1>

      <!-- Form -->
      <div class="bg-white p-6 rounded-xl shadow mb-6">
        <div class="grid md:grid-cols-4 gap-4">
          <input
            v-model="description"
            type="text"
            placeholder="Description"
            class="border rounded-lg p-3"
          />

          <input
            v-model.number="amount"
            type="number"
            placeholder="Amount"
            class="border rounded-lg p-3"
          />

          <select v-model="type" class="border rounded-lg p-3">
            <option value="income">Income</option>
            <option value="expense">Expense</option>
          </select>

          <button
            @click="addTransaction"
            class="bg-blue-600 text-white rounded-lg px-4 py-3 hover:bg-blue-700"
          >
            Add Transaction
          </button>
        </div>
      </div>

      <!-- Summary -->
      <div class="grid md:grid-cols-4 gap-4 mb-6">
        <div class="bg-green-100 p-4 rounded-xl">
          <h3 class="font-semibold">Income</h3>
          <p class="text-2xl">${{ totalIncome }}</p>
        </div>

        <div class="bg-red-100 p-4 rounded-xl">
          <h3 class="font-semibold">Expenses</h3>
          <p class="text-2xl">${{ totalExpenses }}</p>
        </div>

        <div class="bg-blue-100 p-4 rounded-xl">
          <h3 class="font-semibold">Balance</h3>
          <p class="text-2xl">${{ balance }}</p>
        </div>

        <div class="bg-yellow-100 p-4 rounded-xl">
          <h3 class="font-semibold">Status</h3>
          <p>{{ budgetStatus }}</p>
        </div>
      </div>

      <!-- Progress -->
      <div class="bg-white p-6 rounded-xl shadow mb-6">
        <h2 class="font-bold mb-3">Budget Usage</h2>

        <div class="w-full bg-gray-200 h-6 rounded-full overflow-hidden">
          <div
            class="bg-red-500 h-full transition-all duration-300"
            :style="{ width: expensePercentage + '%' }"
          ></div>
        </div>

        <p class="mt-2">{{ expensePercentage.toFixed(0) }}%</p>
      </div>

      <!-- Filter -->
      <div class="mb-4">
        <select v-model="filterType" class="border rounded-lg p-2">
          <option value="all">All</option>
          <option value="income">Income</option>
          <option value="expense">Expense</option>
        </select>
      </div>

      <!-- Transactions -->
      <div class="bg-white rounded-xl shadow overflow-hidden">
        <div class="flex justify-between items-center p-4 border-b">
          <h2 class="font-bold text-lg">Transactions</h2>

          <button
            @click="clearAll"
            class="bg-gray-700 text-white px-4 py-2 rounded-lg"
          >
            Clear All
          </button>
        </div>

        <table class="w-full">
          <thead class="bg-gray-100">
            <tr>
              <th class="p-3">Description</th>
              <th class="p-3">Amount</th>
              <th class="p-3">Type</th>
              <th class="p-3">Action</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="transaction in filteredTransactions"
              :key="transaction.id"
              class="border-t"
            >
              <td class="p-3">
                {{ transaction.desc }}
              </td>

              <td class="p-3">${{ transaction.amount }}</td>

              <td class="p-3">
                <span
                  :class="
                    transaction.type === 'income'
                      ? 'text-green-600 font-medium'
                      : 'text-red-600 font-medium'
                  "
                >
                  {{ transaction.type }}
                </span>
              </td>

              <td class="p-3">
                <button
                  @click="deleteTransaction(transaction.id)"
                  class="bg-red-500 text-white px-3 py-1 rounded"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Category Summary -->
      <div class="bg-white p-6 rounded-xl shadow mt-6">
        <h2 class="font-bold mb-3">Category Summary</h2>

        <ul class="list-disc ml-6">
          <li v-for="(value, key) in categorySummary" :key="key">
            {{ key }} : ${{ value }}
          </li>
        </ul>
      </div>

      <!-- Notifications -->
      <div class="bg-white p-6 rounded-xl shadow mt-6">
        <h2 class="font-bold mb-3">Notifications</h2>

        <ul class="list-disc ml-6">
          <li v-for="(note, index) in notificationLog" :key="index">
            {{ note }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
