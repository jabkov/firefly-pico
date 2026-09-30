<template>
  <van-cell-group inset class="display-flex-column mt-3">
    <transaction-split-header :index="props.splitIndex" :total="transactions.length" style="order: -1" @remove="onRemoveSplit(props.splitIndex)" />

    <transaction-amount-field
      v-model:amount="amount"
      v-model:amount-foreign="amountForeign"
      v-model:currency-foreign="currencyForeign"
      :currency="sourceCurrency"
      :is-foreign-amount-visible="isForeignAmountVisible"
      :name="`amount-${props.splitIndex}`"
      :style="getStyleForField(transactionFormField.amount)"
      :is-amount-required="true"
    />

    <account-select
      v-model="accountSource"
      :label="$t('transaction.source_account')"
      :allowed-types="accountSourceAllowedTypes"
      :style="getStyleForField(transactionFormField.sourceAccount)"
      v-bind="accountSourceBinding"
    />

    <account-select
      v-model="accountDestination"
      :label="$t('transaction.destination_account')"
      :allowed-types="accountDestinationAllowedTypes"
      :style="getStyleForField(transactionFormField.destinationAccount)"
      v-bind="accountDestinationBinding"
    />

    <category-select v-if="profileStore.categoriesEnabled" v-model="category" :style="getStyleForField(transactionFormField.category)" />

    <app-field
      v-model="description"
      :label="$t('description')"
      :name="`description-${props.splitIndex}`"
      type="textarea"
      rows="1"
      autosize
      :icon="TablerIconConstants.fieldText2"
      placeholder="Description"
      :rules="[rule.required()]"
      required
      :style="getStyleForField(transactionFormField.description)"
    />

    <tag-select v-if="profileStore.tagsEnabled" v-model="tags" :style="getStyleForField(transactionFormField.tags)" />

    <transaction-note-field v-model="notes" :style="getStyleForField(transactionFormField.notes)" />

    <budget-select v-if="profileStore.budgetsEnabled" v-model="budget" :style="getStyleForField(transactionFormField.budget)" />
  </van-cell-group>
</template>

<script setup>
import TablerIconConstants from '~/constants/TablerIconConstants'
import TransactionSplitHeader from '~/components/transaction/transaction-split-header.vue'
import TransactionNoteField from '~/components/transaction/transaction-note-field.vue'
import { transactionFormField } from '~/constants/TransactionConstants.js'
import { rule } from '~/utils/ValidationUtils.js'
import { useTransactionForm } from '~/composables/useTransactionForm.js'

const props = defineProps({
  splitIndex: {
    type: Number,
    required: true,
  },
})

const item = defineModel({
  type: Object,
  required: true,
})

const profileStore = useProfileStore()
const itemId = computed(() => item.value?.id)

const {
  amount,
  amountForeign,
  tags,
  description,
  notes,
  budget,
  accountSource,
  accountDestination,
  category,
  currencyForeign,
  transactions,
  onRemoveSplit,
  accountSourceAllowedTypes,
  accountDestinationAllowedTypes,
  sourceCurrency,
  isForeignAmountVisible,
  getStyleForField,
  accountSourceBinding,
  accountDestinationBinding,
} = useTransactionForm({ item, itemId, profileStore, splitIndex: props.splitIndex })
</script>
