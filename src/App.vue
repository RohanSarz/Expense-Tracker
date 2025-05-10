<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expenses="+expenses" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="handleTransactionDeleted"
    />
    <AddTransaction
      @addTransaction="handleTransactionSubmitted"
    />
  </div>
</template>

<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";
import { ref, computed } from "vue";
import { useToast } from "vue-toastification";
const toast = useToast();

const transactions = ref([]);

// accumulating all the transaction amount
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});
// income and expense
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0);
});

const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0);
});

// add transaction
const handleTransactionSubmitted = (transactionData) => {
  transactionData.amount = +transactionData.amount;

  if (
    !transactionData.text ||
    isNaN(transactionData.amount) ||
    transactionData.amount === 0
  ) {
    toast.error("Please enter a valid transaction amount");
    return;
  }

  // Capitalize first letter (optional uniformity)
  const formattedText =
    transactionData.text.charAt(0).toUpperCase() +
    transactionData.text.slice(1);

  const existing = transactions.value.find((t) => t.text === formattedText);

  transactions.value.push({
    id: existing ? existing.id : generateUniqueId(),
    text: formattedText,
    amount: transactionData.amount,
  });

  toast.success("Transaction added successfully");
};

// generate unique id
const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000000);
};
// delete transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );
};
</script>
