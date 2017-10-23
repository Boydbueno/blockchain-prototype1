<template>
  <div>

    <section class="account">
      <header>
        <select v-if="accounts.length > 1" v-model="account">
          <option v-for="account in accounts" :value="account">
            {{ account }}
          </option>
        </select>
        <template v-else>{{ account }}</template>
      </header>
      Ether: {{ ether }} <br />
    </section>

    <section class="contract">
      <header>{{ contractProps.address }}</header>
      <section class="info">
        Beneficiary: {{ contractProps.beneficiary }} <br />
        Goal: {{ contractProps.goal }} <br />
        Value: {{ contractProps.value }} <br />
      </section>

      <section class="actions">
        <input type="button" name="fund" value="fund 0.01 ether" @click="fundOneEther">
        <input type="button" name="withdraw" value="withdraw all your ether" @click="withdrawAll">
        <input type="button" name="claim" value="claim all ether" @click="claim">
      </section>
    </section>

  </div>
</template>

<script>
import Web3 from 'web3'

const contractAddress = '0x12340385532F2c1da854326B09DE458fC20FfF33'
const abi = [
  {
    'constant': true,
    'inputs': [],
    'name': 'beneficiary',
    'outputs': [{
      'name': '',
      'type': 'address',
      'value': '0xfe49332023959dbd91b9ec78267dbecd6fae9410'
    }],
    'payable': false,
    'stateMutability': 'view',
    'type': 'function'
  },
  {
    'constant': false,
    'inputs': [],
    'name': 'withdraw',
    'outputs': [],
    'payable': false,
    'stateMutability': 'nonpayable',
    'type': 'function'
  },
  {
    'constant': true,
    'inputs': [],
    'name': 'value',
    'outputs': [{
      'name': '',
      'type': 'uint256',
      'value': '3000000000000000000'
    }],
    'payable': false,
    'stateMutability': 'view',
    'type': 'function'
  },
  {
    'constant': true,
    'inputs': [],
    'name': 'goal',
    'outputs': [{
      'name': '',
      'type': 'uint256',
      'value': '10000000000000000000'
    }],
    'payable': false,
    'stateMutability': 'view',
    'type': 'function'
  },
  {
    'constant': false,
    'inputs': [],
    'name': 'claim',
    'outputs': [],
    'payable': false,
    'stateMutability': 'nonpayable',
    'type': 'function'
  },
  {
    'constant': true,
    'inputs': [{
      'name': '',
      'type': 'address'
    }],
    'name': 'balanceOf',
    'outputs': [{
      'name': '',
      'type': 'uint256',
      'value': '0'
    }],
    'payable': false,
    'stateMutability': 'view',
    'type': 'function'
  },
  {
    'constant': true,
    'inputs': [],
    'name': 'isClosed',
    'outputs': [{
      'name': '',
      'type': 'bool',
      'value': false
    }],
    'payable': false,
    'stateMutability': 'view',
    'type': 'function'
  },
  {
    'inputs': [
      {
        'name': 'ifSuccessfulSendTo',
        'type': 'address',
        'index': 0,
        'typeShort': 'address',
        'bits': '',
        'displayName': 'if Successful Send To',
        'template': 'elements_input_address',
        'value': '0xFe49332023959Dbd91B9ec78267dBECd6Fae9410'
      },
      {
        'name': 'goalInEther',
        'type': 'uint256',
        'index': 1,
        'typeShort': 'uint',
        'bits': '256',
        'displayName': 'goal In Ether',
        'template': 'elements_input_uint',
        'value': '10'
      }
    ],
    'payable': false,
    'stateMutability': 'nonpayable',
    'type': 'constructor'
  },
  {
    'payable': true,
    'stateMutability': 'payable',
    'type': 'fallback'
  },
  {
    'anonymous': false,
    'inputs': [
      {
        'indexed': false,
        'name': 'withdrawer',
        'type': 'address'
      },
      {
        'indexed': false,
        'name': 'amount',
        'type': 'uint256'
      }
    ],
    'name': 'FundWithdrew',
    'type': 'event'
  },
  {
    'anonymous': false,
    'inputs': [
      {
        'indexed': false,
        'name': 'backer',
        'type': 'address'
      },
      {
        'indexed': false,
        'name': 'amount',
        'type': 'uint256'
      }
    ],
    'name': 'FundAdded',
    'type': 'event'
  }
]

export default {
  name: 'Home',

  data () {
    return {
      useLocalNode: false,
      accounts: [],
      account: null,
      wei: null,
      contractProps: {
        beneficiary: 0,
        goal: 0,
        value: 0,
        address: 0
      }
      // contract, reactivity breaks assignment
    }
  },

  computed: {
    ether () {
      return window.web3.utils.fromWei(this.wei || 0, 'ether')
    }
  },

  created () {
    if (Web3.givenProvider && !this.useLocalNode) {
      window.web3 = new Web3(Web3.givenProvider)
      console.log('Given provider', Web3.givenProvider)
    } else {
      window.web3 = new Web3('ws://localhost:8546')
      console.log('Connected to own node')
    }

    window.web3.eth.getAccounts().then((accounts) => {
      this.accounts = accounts
      this.account = accounts[0]

      return window.web3.eth.getBalance(this.account)
    }).then((result) => {
      this.wei = result
    }).catch((error) => {
      console.log(error)
    })

    this.contract = new window.web3.eth.Contract(abi, contractAddress)
    this.contractProps.address = contractAddress

    this.contract.events.FundAdded({}, (error, result) => {
      if (error) {
        console.error(error)
        // We probably got here because the current provider doesn't support subscription
        // So we might need some polling fallback
      }
    }).on('data', (log) => {
      this.contractProps.value = parseInt(this.contractProps.value) + parseInt(log.returnValues.amount)
      console.log(log)
    }).on('error', (error) => {
      console.log(error)
    })

    this.contract.events.FundWithdrew({}, (error, result) => {
      if (error) {
        console.error(error)
        // We probably got here because the current provider doesn't support subscription
        // So we might need some polling fallback
      }
    }).on('data', (log) => {
      this.contractProps.value = parseInt(this.contractProps.value) - parseInt(log.returnValues.amount)
      console.log(log)
    }).on('error', (error) => {
      console.log(error)
    })

    this._updateContract()
  },

  methods: {

    fundOneEther () {
      window.web3.eth.sendTransaction({
        from: this.account,
        to: contractAddress,
        value: '10000000000000000'
      }).on('transactionHash', (hash) => {
        console.log(hash)
      }).on('receipt', (receipt) => {
        console.log(receipt)
      }).on('confirmation', (number, object) => {
        console.log(number)
        console.log(object)
      }).on('error', (error) => {
        console.log(error)
      })
    },

    withdrawAll () {
      this.contract.methods.withdraw().send({
        from: this.account
      }).on('error', (error) => {
        console.log(error)
      })
    },

    claim () {
      this.contract.methods.claim().send({
        from: this.account
      }).on('error', (error) => {
        console.log(error)
      })
    },

    _updateContract () {
      this.contract.methods.beneficiary().call().then((result) => {
        this.contractProps.beneficiary = result
      })

      this.contract.methods.goal().call().then((result) => {
        this.contractProps.goal = result
      })

      this.contract.methods.value().call().then((result) => {
        this.contractProps.value = result
      })
    }

  }
}
</script>

<style scoped>

</style>
