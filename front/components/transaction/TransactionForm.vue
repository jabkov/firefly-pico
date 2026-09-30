<template>
  <transaction-type-tabs v-model="type" class="mx-3 mt-1 mb-1" />

  <van-form ref="formRef" :disabled="props.disabled" :name="props.formName" class="transaction-form-group" @submit="emit('submit')" @failed="emit('failed', $event)">
    <van-cell-group v-if="isSplitTransaction" inset class="mb-3">
      <app-field
        v-model="groupTitle"
        :label="$t('transaction.group_title')"
        name="group_title"
        type="textarea"
        rows="1"
        autosize
        :icon="TablerIconConstants.fieldText1"
        :rules="[rule.required()]"
        required
      />
    </van-cell-group>

    <van-cell-group inset class="dynamic-masonry display-flex-column">
      <transaction-split-header v-if="isSplitTransaction" :index="0" :total="transactions.length" style="order: -1" @remove="onRemoveSplit(0)" />

      <transaction-amount-field
        v-model:amount="amount"
        v-model:amount-foreign="amountForeign"
        v-model:currency-foreign="currencyForeign"
        :currency="sourceCurrency"
        :is-foreign-amount-visible="isForeignAmountVisible"
        name="amount"
        :style="getStyleForField(transactionFormField.amount)"
        :is-amount-required="true"
      />

      <account-select
        v-model="accountSource"
        :label="$t('transaction.source_account')"
        :allowed-types="accountSourceAllowedTypes"
        :style="getStyleForField(transactionFormField.sourceAccount)"
        v-bind="accountSourceBinding"
      >
        <template #label>
          <div class="flex-center-vertical gap-1">
            <div class="flex-1">{{ $t('transaction.source_account') }}</div>
            <van-button v-if="showSourceAccountSuggestion" size="mini" class="suggestion-button cursor-pointer" @click="navigateTo(RouteConstants.ROUTE_SETTINGS_TRANSACTION_DEFAULT_FORM_VALUES)">
              Set your default
            </van-button>
          </div>
        </template>
      </account-select>

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
        name="description"
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

      <div :style="getStyleForField(transactionFormField.date)">
        <app-date-time-grid v-model="date" name="date" :rules="[rule.required()]" required />

        <div class="px-3 flex-center-vertical gap-1">
          <van-button size="small" class="cursor-pointer" @click="onSubDay">{{ $t('sub_day') }}</van-button>
          <van-button size="small" class="cursor-pointer" @click="onToday">{{ $t('today') }}</van-button>
          <van-button size="small" class="cursor-pointer" @click="onAddDay">{{ $t('add_day') }}</van-button>
        </div>
      </div>

      <app-date
        v-for="extraDateField in transactionExtraDateFieldList"
        :key="extraDateField.code"
        v-model="extraDates[extraDateField.code].value"
        :label="$t(extraDateField.t)"
        :icon="extraDateField.icon"
        :style="getStyleForField(extraDateField)"
      />

      <transaction-note-field v-model="notes" :style="getStyleForField(transactionFormField.notes)" />

      <budget-select v-if="profileStore.budgetsEnabled" v-model="budget" :style="getStyleForField(transactionFormField.budget)" />

      <piggy-bank-select v-if="profileStore.piggyBanksEnabled && isTypeTransfer && !itemId" v-model="piggyBank" :style="getStyleForField(transactionFormField.piggyBank)" />

      <transaction-attachments-list :transaction="item" :style="getStyleForField(transactionFormField.attachments)" />
    </van-cell-group>

    <transaction-split-form v-for="(split, index) in transactions.slice(1)" :key="index + 1" v-model="item" :split-index="index + 1" />

    <div class="mx-3 mt-3">
      <van-button block plain hairline icon="plus" class="cursor-pointer" @click="onAddSplit">{{ $t('transaction.add_split') }}</van-button>
    </div>

    <slot name="actions" :is-split-transaction="isSplitTransaction" :is-type-transfer="isTypeTransfer" />
  </van-form>
</template>

<script setup>
import RouteConstants from '~/constants/RouteConstants'
import Transaction from '~/models/Transaction'
import TablerIconConstants from '~/constants/TablerIconConstants'
import TransactionSplitHeader from '~/components/transaction/transaction-split-header.vue'
import TransactionSplitForm from '~/components/transaction/transaction-split-form.vue'
import TransactionAttachmentsList from '~/components/transaction/transaction-attachements/transaction-attachments-list.vue'
import TransactionNoteField from '~/components/transaction/transaction-note-field.vue'
import { transactionFormField, transactionExtraDateFieldList } from '~/constants/TransactionConstants.js'
import { rule } from '~/utils/ValidationUtils.js'
import { useTransactionForm } from '~/composables/useTransactionForm.js'
import { getGUID } from '~/utils/Utils.js'

const props = defineProps({
  formName: {
    type: String,
    default: () => `form-${getGUID()}`,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['submit', 'failed'])
const item = defineModel({
  type: Object,
  default: () => new Transaction().getEmpty(),
})

const profileStore = useProfileStore()
const formRef = ref(null)
const itemId = computed(() => item.value?.id)

const {
  amount,
  amountForeign,
  date,
  extraDates,
  tags,
  description,
  notes,
  budget,
  piggyBank,
  accountSource,
  accountDestination,
  category,
  type,
  currencyForeign,
  transactions,
  isSplitTransaction,
  groupTitle,
  onAddSplit,
  onRemoveSplit,
  accountSourceAllowedTypes,
  accountDestinationAllowedTypes,
  sourceCurrency,
  isForeignAmountVisible,
  applyAssistantTransaction,
  onSubDay,
  onToday,
  onAddDay,
  isTypeTransfer,
  getStyleForField,
  accountSourceBinding,
  accountDestinationBinding,
  showSourceAccountSuggestion,
} = useTransactionForm({ item, itemId, profileStore })

const validate = async () => {
  return formRef.value?.validate()
}

const submit = () => {
  formRef.value?.submit()
}

defineExpose({
  validate,
  submit,
  applyAssistantTransaction,
  isSplitTransaction,
})
</script>
