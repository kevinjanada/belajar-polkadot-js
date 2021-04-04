<template>
  <div id="app">
    Balance: {{ balanceFree }}
    nonce: {{ nonce }}
  </div>
</template>

<script>
import { mnemonicGenerate } from '@polkadot/util-crypto'
import { Keyring } from '@polkadot/keyring';
import { ApiPromise, WsProvider } from '@polkadot/api';
import types from './types.json'

export default {
  name: 'App',
  components: {
  },
  data: () => ({
    mnemonic: '',
    pair: null,
    api: null,
    balanceFree: null,
    nonce: 0,
  }),
  async mounted() {
    this.generateAccount()
    await this.connectToBlockchain()
    await this.getBalance(
      '5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY'
    )
  },
  methods: {
    generateAccount() {
      const keyring = new Keyring({ type: 'ed25519', ss58Format: 42 })

      const mnemonic = mnemonicGenerate()
      this.mnemonic = mnemonic

      const pair = keyring.addFromUri(mnemonic, { name: 'first pair' }, 'ed25519');
      this.pair = pair

      console.log(this.pair)
    },
    async connectToBlockchain() {
      const wsProvider = new WsProvider('wss://debio.theapps.dev/node');
      const api = await ApiPromise.create({
        provider: wsProvider,
        types: types
      });
      this.api = api
    },
    async getBalance(addr) {
      const { nonce, data: balance } = await this.api.query.system.account(addr);
      console.log(nonce)
      console.log(balance)
      this.nonce = nonce
      this.balanceFree = balance.free
      console.log(`balance of ${balance.free} and a nonce of ${nonce}`);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
