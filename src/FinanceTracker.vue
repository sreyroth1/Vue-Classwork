<template>
  <div class="container">
    <h1>Personal Finance Tracker</h1>

    <div class="form">
      <input
        type="text"
        v-model="description"
        placeholder="Description"
      />

      <input
        type="number"
        v-model.number="amount"
        placeholder="Amount"
      />

      <select v-model="type">
        <option value="income">Income</option>
        <option value="expense">Expense</option>
      </select>

      <button @click="addTransaction">
        Add
      </button>
    </div>

    <h2>Summary</h2>

    <p>Total Income: ${{ totalIncome }}</p>
    <p>Total Expense: ${{ totalExpenses }}</p>
    <p>Balance: ${{ balance }}</p>
    <p>Expense Percentage: {{ expensePercentage }}%</p>
    <p>Status: {{ budgetStatus }}</p>

    <h3 v-if="isOverBudget" style="color:red">
      ⚠ Over Budget!
    </h3>

    <label>Filter:</label>

    <select v-model="filterType">
      <option value="all">All</option>
      <option value="income">Income</option>
      <option value="expense">Expense</option>
    </select>

    <ul>
      <li
        v-for="item in filteredTransactions"
        :key="item.id"
      >
        {{ item.description }}
        -
        ${{ item.amount }}
        -
        {{ item.type }}

        <button @click="deleteTransaction(item.id)">
          Delete
        </button>
      </li>
    </ul>

    <button @click="clearAll">
      Clear All
    </button>

    <h2>Category Summary</h2>

    <div v-for="(value,key) in categorySummary" :key="key">
      {{ key }} : ${{ value }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";

interface Transaction {
  id: number;
  description: string;
  amount: number;
  type: "income" | "expense";
  date: string;
}

// Reactive State

const transactions = ref<Transaction[]>([]);

const filterType = ref<"all" | "income" | "expense">(
  "all"
);

const budgetLimit = ref<number>(1000);

// Form

const description = ref<string>("");
const amount = ref<number>(0);
const type = ref<"income" | "expense">("income");

// Methods

const addTransaction = () => {
  if (!description.value || amount.value <= 0) return;

  transactions.value.push({
    id: Date.now(),
    description: description.value,
    amount: amount.value,
    type: type.value,
    date: new Date().toLocaleDateString(),
  });

  description.value = "";
  amount.value = 0;
  type.value = "income";
};

const deleteTransaction = (id: number) => {
  transactions.value = transactions.value.filter(
    (item) => item.id !== id
  );
};

const clearAll = () => {
  transactions.value = [];
};

// Computed Properties

const filteredTransactions = computed(() => {
  if (filterType.value === "all") {
    return transactions.value;
  }

  return transactions.value.filter(
    (item) => item.type === filterType.value
  );
});

const totalIncome = computed(() => {
  return transactions.value
    .filter((item) => item.type === "income")
    .reduce((sum, item) => sum + item.amount, 0);
});

const totalExpenses = computed(() => {
  return transactions.value
    .filter((item) => item.type === "expense")
    .reduce((sum, item) => sum + item.amount, 0);
});

const balance = computed(() => {
  return totalIncome.value - totalExpenses.value;
});

const isOverBudget = computed(() => {
  return totalExpenses.value > budgetLimit.value;
});

const expensePercentage = computed(() => {
  if (budgetLimit.value === 0) return 0;

  const percent =
    (totalExpenses.value / budgetLimit.value) * 100;

  return Math.min(percent, 100).toFixed(2);
});

const categorySummary = computed(() => {
  return {
    income: totalIncome.value,
    expense: totalExpenses.value,
  };
});

const budgetStatus = computed(() => {
  if (isOverBudget.value) {
    return "Over Budget";
  }

  return "Within Budget";
});

// Watchers

watch(
  transactions,
  (newValue) => {
    localStorage.setItem(
      "transactions",
      JSON.stringify(newValue)
    );
  },
  { deep: true }
);

watch(balance, (newValue) => {
  if (newValue < 0) {
    alert("Warning: Your balance is negative!");
  }
});

watch(isOverBudget, (newValue) => {
  if (newValue) {
    console.log("You exceeded your budget!");
  }
});

// Load data from localStorage

const saved = localStorage.getItem("transactions");

if (saved) {
  transactions.value = JSON.parse(saved);
}
</script>

<style scoped>
.container {
  width: 700px;
  margin: auto;
  font-family: Arial;
}

.form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input,
select,
button {
  padding: 8px;
}
</style>