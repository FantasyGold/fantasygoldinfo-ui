<template>
  <section class="container">
    <div class="card section-card">
      <div class="card-header">
        <div class="card-header-icon">
          <Icon icon="code" fixedWidth />
        </div>
        <h3 class="card-header-title">{{ $t('contract.summary') }}</h3>
      </div>
      <div class="card-body info-table">
        <div class="columns">
          <div class="column info-title">{{ $t('contract.address') }}</div>
          <div class="column info-value">
            <AddressLink :address="addressHex" plain />
          </div>
        </div>
        <template v-if="fgc20">
          <div class="columns" v-if="fgc20.name">
            <div class="column info-title">{{ $t('contract.token.name') }}</div>
            <div class="column info-value">{{ fgc20.name }}</div>
          </div>
          <div class="columns" v-if="fgc20.holders">
            <div class="column info-title">{{ $t('contract.token.total_supply') }}</div>
            <div class="column info-value monospace">
              {{ fgc20.totalSupply | fgc20(fgc20.decimals, true) }}
              {{ fgc20.symbol || $t('contract.token.tokens') }}
            </div>
          </div>
          <div class="columns">
            <div class="column info-title">{{ $t('contract.token.token_holders') }}</div>
            <div class="column info-value">{{ fgc20.holders }}</div>
          </div>
        </template>
        <template v-if="fgc721">
          <div class="columns" v-if="fgc721.name">
            <div class="column info-title">{{ $t('contract.token.name') }}</div>
            <div class="column info-value">{{ fgc721.name }}</div>
          </div>
          <div class="columns">
            <div class="column info-title">{{ $t('contract.token.total_supply') }}</div>
            <div class="column info-value monospace">
              {{ fgc721.totalSupply }}
              {{ fgc721.symbol || $t('contract.token.tokens') }}
            </div>
          </div>
        </template>
        <div class="columns">
          <div class="column info-title">{{ $t('contract.balance') }}</div>
          <div class="column info-value monospace">{{ balance | fantasygold }} FGC</div>
        </div>
        <div class="columns">
          <div class="column info-title">{{ $t('contract.total_received') }}</div>
          <div class="column info-value monospace">{{ totalReceived | fantasygold }} FGC</div>
        </div>
        <div class="columns">
          <div class="column info-title">{{ $t('contract.total_sent') }}</div>
          <div class="column info-value monospace">{{ totalSent | fantasygold }} FGC</div>
        </div>
        <div class="columns" v-if="existingTokenBalances.length">
          <div class="column info-title">{{ $t('address.token_balances') }}</div>
          <div class="column info-value">
            <div v-for="token in existingTokenBalances" class="monospace">
              {{ token.balance | fgc20(token.decimals) }}
              <AddressLink :address="token.addressHex">
                {{ token.symbol || token.name || $t('contract.token.tokens') }}
              </AddressLink>
            </div>
          </div>
        </div>
        <div class="columns">
          <div class="column info-title">{{ $t('contract.transaction_count') }}</div>
          <div class="column info-value">{{ transactionCount }}</div>
        </div>
      </div>
    </div>

    <div v-if="transactionCount" class="tabs is-centered">
      <ul>
        <li :class="{'is-active': $route.matched.some(route => route.name === 'contract-id')}">
          <nuxt-link :to="{name: 'contract-id', params: {id}}">
            {{ $t('contract.transaction_list') }}
          </nuxt-link>
        </li>
        <li
          v-if="type === 'fgc20'"
          :class="{'is-active': $route.matched.some(route => route.name === 'contract-id-rich-list')}">
          <nuxt-link :to="{name: 'contract-id-rich-list', params: {id}}">
            {{ $t('misc.rich_list_title') }}
          </nuxt-link>
        </li>
      </ul>
    </div>
    <nuxt-child :fgc20="fgc20" />
  </section>
</template>

<script>
  import Contract from '@/models/contract'
  import {RequestError} from '@/services/fantasygoldinfo-api'

  export default {
    head() {
      return {
        title: this.$t('blockchain.contract') + ' ' + this.id
      }
    },
    data() {
      return {
        address: '',
        addressHex: '',
        vm: '',
        type: '',
        fgc20: null,
        fgc721: null,
        balance: '0',
        totalReceived: '0',
        totalSent: '0',
        fgc20Balances: [],
        transactionCount: 0
      }
    },
    async asyncData({req, params, query, redirect, error}) {
      try {
        let contract = await Contract.get(params.id)
        return {
          address: contract.address,
          addressHex: contract.addressHex,
          vm: contract.vm,
          type: contract.type,
          fgc20: contract.fgc20,
          fgc721: contract.fgc721,
          balance: contract.balance,
          totalReceived: contract.totalReceived,
          totalSent: contract.totalSent,
          fgc20Balances: contract.fgc20Balances,
          transactionCount: contract.transactionCount
        }
      } catch (err) {
        if (err instanceof RequestError) {
          if (err.code === 404) {
            error({statusCode: 404, message: `Contract ${param.id} not found`})
          } else {
            error({statusCode: err.code, message: err.message})
          }
        } else {
          error({statusCode: 500, message: err.message})
        }
      }
    },
    computed: {
      id() {
        return this.$route.params.id
      },
      pages() {
        return Math.ceil(this.transactionCount / 20)
      },
      existingTokenBalances() {
        return this.fgc20Balances.filter(token => token.balance !== '0')
      }
    }
  }
</script>
