<template>
  <div>
    Coinbase: {{ coinbase }} <br />
    Ether: {{ ether }} <br />
    <input type="button" name="fund" value="fund 1 ether" @click="fundOneEther">
    <input type="button" name="withdraw" value="withdraw all ether" @click="withdrawAll">
  </div>
</template>

<script>
import Web3 from 'web3'
// import Contract from 'web3-eth-contract'

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
      coinbase: null,
      wei: null
      // contract, reactivity breaks assignment
    }
  },

  computed: {
    ether () {
      if (this.wei === null) return 0
      return window.web3.utils.fromWei(this.wei, 'ether')
    }
  },

  created () {
    window.web3 = new Web3(window.web3.currentProvider)

    window.web3.eth.getCoinbase().then((result) => {
      this.coinbase = result

      return window.web3.eth.getBalance(this.coinbase).then((result) => {
        this.wei = result.toString()
      })
    })

    this.contract = new window.web3.eth.Contract(abi, contractAddress)
  },

  methods: {

    fundOneEther () {
      window.web3.eth.sendTransaction({
        from: this.coinbase,
        to: contractAddress,
        value: '1000000000000000000'
      })
    },

    withdrawAll () {
      this.contract.methods.withdraw().send({from: this.coinbase})
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
