<template lang="pug">
  div.wallet
    .table-header
      el-input(v-model='search' prefix-icon="el-icon-search" placeholder="Search name or paste address" size="small" clearable)
      el-checkbox() Hide small balances
    .el-card.is-always-shadow
      el-table.alcor-table.noHover(
        :data='balances',
        style='width: 100%',
        :default-sort='{ prop: "weekVolume", order: "descending" }'
      )
        el-table-column(label='Asset', prop='date', :width='isMobile ? 150 : 280')
          template(slot-scope='{row}')
            .asset-container
              TokenImage(
                :src='$tokenLogo(row.currency, row.contract)',
                :height="isMobile? '20' : '30'"
              )

              div.asset
                span.asset-name {{ row.currency }}
                span.asset-contract.cancel {{ row.contract }}

        el-table-column(
          label='Total',
          sort-by='amount',
          sortable,
        )
          template(slot-scope='{row}')
            div.amount-val
              .amount {{ row.amount | commaFloat(4, row.decimals) }}
              .val.cancel = ${{ row.usd_value }}
        el-table-column(
          label='Available',
          sort-by='amount',
          sortable,
        )
          template(slot-scope='{row}')
            div.amount-val
              .amount {{ row.amount | commaFloat(4, row.decimals) }}
              .val.cancel = ${{ row.usd_value }}
        //el-table-column(label='In Order')
          //- TODO: dynamic
          template(slot-scope='{row}')
            div.amount-val
              .amount {{row.amount}}
              .val.cancel(v-if="row.contract == network.baseToken.contract") = ${{ $systemToUSD(row.amount) }}
              .val.cancel(v-else) = $0.00
        //el-table-column(label='WAX Value')
          template(slot-scope='{row}')
            div.amount-val
              .amount {{row.amount}}
              .val.cancel(v-if="row.contract == network.baseToken.contract") = ${{ $systemToUSD(row.amount) }}
              .val.cancel(v-else) = $0.00
        el-table-column(
          label='Actions',
          width="260"
        )
          template(slot-scope='{row}')
            .actions
              el-button(type="text" @click="openDeposit").hover-opacity Deposit
              el-button(type="text" @click="openWithdraw(row)").hover-opacity Transfer
              el-button(type="text" @click="pools(row)").hover-opacity Pools
              el-button.hover-opacity(type="text" @click="trade(row)") Trade
    DepositPopup(ref="depositPopup")
    WithdrawPopup(ref="withdrawPopup")
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import TokenImage from '@/components/elements/TokenImage'
import DepositPopup from '@/components/wallet/DepositPopup'
import WithdrawPopup from '@/components/wallet/WithdrawPopup'
export default {
  name: 'Wallet',
  components: {
    TokenImage,
    DepositPopup,
    WithdrawPopup
  },
  data: () => ({
    search: ''
  }),
  computed: {
    ...mapGetters(['user']),
    ...mapState(['network', 'markets']),

    balances() {
      if (!this.user) return []
      if (!this.user.balances) return []

      return this.user.balances
        .filter((b) => {
          if (parseFloat(b.amount) == 0) return false

          return b.id.toLowerCase().includes(this.search.toLowerCase())
        })
        .sort((a, b) =>
          a.contract == this.network.baseToken.contract ? -1 : 1
        )
    }
  },

  methods: {
    pools(token) {
      this.$router.push({
        name: 'swap',
        //query: { input: token.id.replace('@', '-') }
      })

      this.$store.commit('swap/setInput', {
        contract: token.contract,
        symbol: token.currency,
        precision: parseFloat(token.decimals)
      })
    },

    trade(token) {
      this.$router.push({
        name: 'markets',
        query: { tab: 'all', search: `${token.currency}-${token.contract}` }
      })
    },
    openDeposit() {
      this.$refs.depositPopup.openPopup({})
    },
    openWithdraw(row) {
      this.$refs.withdrawPopup.openPopup({
        token: {
          contract: row.contract,
          currency: row.currency,
          decimals: Number(row.decimals)
        }
      })
    }
  }
}
</script>

<style scoped lang="scss">
.table-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  margin-bottom: 10px;
  .el-input {
    max-width: 300px;
    margin-right: 8px;
    margin-bottom: 8px;
  }
  .el-input__inner {
    background: transparent !important;
  }
}
.el-card {
  border: none;
}
.asset-container {
  display: flex;
  align-items: center;
  .asset {
    display: flex;
    flex-direction: column;
    margin-left: 10px;
  }
  .asset-name {
    font-weight: bold;
  }
}
.amount-val {
  .amount {
    color: var(--text-default);
    font-size: 0.9rem;
  }
  .val {
    color: var(--cancel);
    font-size: 0.8rem;
  }
}
.actions {
  display: flex;
  .el-button--text {
    color: var(--main-green) !important;
    font-weight: 400;
  }
}
</style>
