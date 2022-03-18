<template>
  <div>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
      <h1 class="h2">Swap now</h1>
    </div>

    <div v-if="isUserConnected">
      <p>Connected to: {{getChainName}}</p>

      <p>Token address A:
        <input v-model="tokena"> 
      </p>
      <p>Token address B:
        <input v-model="tokenb">
      </p>
      <a href="#" @click="select">Confirm tokens</a>
      <div v-if="confirmed">
        <p>Your balance token A is: {{balance}}</p>
        <div v-if="!nosuch">
          <p>Input token A amount to swap:
            <input v-model="amount">
          </p>
          <p>Price is: {{price}} </p>
          <p>You will receive ~ {{receive}} </p>
          <a href="#" @click="approve">Approve Token A</a><br>
          <a href="#" @click="swap">Swap Token A -> Token B</a>
        </div>
        <div v-else>
          No such pair. Try to  <router-link to="/add">add liquidity first</router-link>
        </div>
        <br/>
      </div>

    </div>
    <div v-else>Please, connect your wallet first</div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { ethers } from "ethers";
const erc20Abi = [
  "function balanceOf(address owner) view returns (uint256)",
  "function approve(address spender, uint256 amount) public returns (bool)"
];
const uniAbi = [
  "function getPair(address tokenA, address tokenB) external view returns (address pair)"
];
const routerAbi = ["function swapExactTokensForTokens(        uint amountIn,        uint amountOutMin,        address[] calldata path,        address to,        uint deadline    ) external returns (uint[] memory amounts)"]
const pairAbi = ["function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast)"];

export default {
  name: "Home",
  data: function () {
     return {
       tokena: "0xa4Aa437E7b1F1ee49ad6CD291c9fdAE37707F5dA",
       tokenb: "0x7db03a15Aa2A1Cd87034FE2E9e08D2e422f51c9A",
       balance: 0,
       amount: 0,
       price:0,
       nosuch: false,
       confirmed: false,
       pair: '0x0000000000000000000000000000000000000000'
     }
  },
  computed: {
    ...mapGetters("accounts", ["getChainName", "isUserConnected", "getActiveAccount", "getProviderEthers"]),
    receive() {
      return this.price * this.amount;
    }
  },
  methods: {
    select: function () {
      this.confirmed = true;
      const contract = new ethers.Contract(this.tokena, erc20Abi, this.getProviderEthers);
      console.log( this.getActiveAccount)
      let that = this;
      contract.functions.balanceOf( that.getActiveAccount).then(function(balance) {
        that.balance = balance/Math.pow(10, 18);
        that.amount = balance/Math.pow(10, 18);
      });
      const uni = new ethers.Contract('0x9bF4D4FE19413d2b6e8f933767726193226392Bb', uniAbi, that.getProviderEthers);
      uni.functions.getPair( that.tokena, that.tokenb).then(function(paircontract) {
        if (paircontract[0] !== '0x0000000000000000000000000000000000000000') {
          that.pair = paircontract[0];
          that.nosuch = false;
          const pair = new ethers.Contract(paircontract[0], pairAbi, that.getProviderEthers);
          pair.functions.getReserves().then(function(reserves) {
            console.log(reserves)
            that.price = reserves.reserve0.toString() / reserves.reserve1.toString();
          })
        } else {
          that.nosuch = true;
        }
      });
      
    },
    approve: async function () {
      const Token1 = new ethers.Contract(this.tokena, erc20Abi, this.getProviderEthers.getSigner())
      const t1Approve = await Token1.approve(this.pair, ethers.constants.MaxUint256)
      await t1Approve.wait()
    },
    swap: async function () {
      
      const Router = new ethers.Contract("0x7740B7a39AED690075E05a5335fcC7C2D6E04311", routerAbi, this.getProviderEthers.getSigner())
      console.log(this.amount)
      console.log(ethers.utils.parseEther(this.amount))
      const trans = await Router.swapExactTokensForTokens(
        ethers.utils.parseEther(this.amount),
        ethers.utils.parseEther('0'),
        [this.tokena, this.tokenb],
        this.getActiveAccount,
        ethers.utils.parseEther('10000000')
      )
      console.log(await trans.wait());

      return trans
    }
  }
}
</script>
