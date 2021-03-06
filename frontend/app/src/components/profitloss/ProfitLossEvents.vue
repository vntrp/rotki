<template>
  <v-card>
    <v-card-title>
      <card-title>{{ $t('profit_loss_events.title') }}</card-title>
    </v-card-title>
    <v-card-text>
      <v-sheet outlined rounded>
        <v-data-table
          :headers="headers"
          :items="indexedEvents"
          single-expand
          class="profit-loss-events__table"
          :expanded.sync="expanded"
          item-key="index"
          sort-by="time"
          sort-desc
          :footer-props="footerProps"
        >
          <template #item.location="{ item }">
            <location-display :identifier="item.location" />
          </template>
          <template #item.time="{ item }">
            <date-display :timestamp="item.time" />
          </template>
          <template #header.paidInProfitCurrency>
            {{ $t('profit_loss_events.headers.paid_in', { currency }) }}
          </template>
          <template #header.taxableBoughtCostInProfitCurrency>
            {{
              $t('profit_loss_events.headers.taxable_bought_cost_in', {
                currency
              })
            }}
          </template>
          <template #header.taxableReceivedInProfitCurrency>
            {{
              $t('profit_loss_events.headers.taxable_received_in', {
                currency
              })
            }}
          </template>
          <template #item.paidInProfitCurrency="{ item }">
            <amount-display :value="item.paidInProfitCurrency" />
          </template>
          <template #item.paidInAsset="{ item }">
            <amount-display
              :value="item.paidInAsset"
              :asset="item.paidAsset ? item.paidAsset : ''"
            />
          </template>
          <template #item.taxableAmount="{ item }">
            <amount-display :value="item.taxableAmount" />
          </template>
          <template #item.taxableBoughtCostInProfitCurrency="{ item }">
            <amount-display :value="item.taxableBoughtCostInProfitCurrency" />
          </template>
          <template #item.receivedInAsset="{ item }">
            <amount-display
              :value="item.receivedInAsset"
              :asset="item.receivedAsset ? item.receivedAsset : ''"
            />
          </template>
          <template #item.taxableReceivedInProfitCurrency="{ item }">
            <amount-display :value="item.taxableReceivedInProfitCurrency" />
          </template>
          <template #item.isVirtual="{ item }">
            <v-icon v-if="item.isVirtual" color="success"> mdi-check</v-icon>
          </template>
          <template #expanded-item="{ headers, item }">
            <td
              :colspan="headers.length"
              class="profit-loss-events__cost-basis"
            >
              <cost-basis-table
                v-if="item.costBasis"
                :cost-basis="item.costBasis"
              />
            </td>
          </template>
          <template #item.expand="{ item }">
            <row-expander
              v-if="item.costBasis"
              :expanded="expanded.includes(item)"
              @click="expanded = expanded.includes(item) ? [] : [item]"
            />
          </template>
        </v-data-table>
      </v-sheet>
    </v-card-text>
  </v-card>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { DataTableHeader } from 'vuetify';
import { mapGetters, mapState } from 'vuex';
import AmountDisplay from '@/components/display/AmountDisplay.vue';
import DateDisplay from '@/components/display/DateDisplay.vue';
import RowExpander from '@/components/helper/RowExpander.vue';
import CostBasisTable from '@/components/profitloss/CostBasisTable.vue';
import { footerProps } from '@/config/datatable.common';
import { ProfitLossEvent } from '@/store/reports/types';

type IndexedProfitLossEvent = ProfitLossEvent & { index: number };

@Component({
  components: {
    CostBasisTable,
    RowExpander,
    DateDisplay,
    AmountDisplay
  },
  computed: {
    ...mapState('reports', ['currency', 'events']),
    ...mapGetters('balances', ['exchangeRate'])
  }
})
export default class ProfitLossEvents extends Vue {
  events!: ProfitLossEvent[];
  currency!: string;
  exchangeRate!: (currency: string) => number;
  expanded = [];

  get indexedEvents(): IndexedProfitLossEvent[] {
    return this.events.map((value, index) => ({
      ...value,
      index
    }));
  }

  readonly headers: DataTableHeader[] = [
    {
      text: this.$t('profit_loss_events.headers.type').toString(),
      value: 'type'
    },
    {
      text: this.$t('profit_loss_events.headers.location').toString(),
      value: 'location',
      width: '120px',
      align: 'center',
      class: 'profit-loss-events__table__header__location'
    },
    {
      text: this.$t('profit_loss_events.headers.paid_in', {
        currency: 'USD'
      }).toString(),
      value: 'paidInProfitCurrency',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.paid_in_asset').toString(),
      value: 'paidInAsset',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.taxable_amount').toString(),
      value: 'taxableAmount',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.taxable_bought_cost_in', {
        currency: 'USD'
      }).toString(),
      value: 'taxableBoughtCostInProfitCurrency',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.received_in_asset').toString(),
      value: 'receivedInAsset',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.taxable_received_in', {
        currency: 'USD'
      }).toString(),
      value: 'taxableReceivedInProfitCurrency',
      align: 'end'
    },
    {
      text: this.$t('profit_loss_events.headers.time').toString(),
      value: 'time'
    },
    {
      text: this.$t('profit_loss_events.headers.virtual').toString(),
      value: 'isVirtual',
      align: 'center'
    },
    {
      text: '',
      value: 'expand',
      align: 'end',
      sortable: false
    }
  ];

  readonly footerProps = footerProps;
}
</script>

<style lang="scss" scoped>
.profit-loss-events {
  &__cost-basis {
    background-color: var(--v-rotki-light-grey-base);
  }
}

::v-deep {
  .profit-loss-events {
    &__table {
      &__header {
        &__location {
          span {
            padding-left: 16px;
          }
        }
      }
    }
  }
}
</style>
