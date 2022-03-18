<template>
  <div>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
      <h1 class="h2">Add liquidity</h1>
    </div>

    <div v-if="isUserConnected">
      <p>Connected to: {{getChainName}}</p>

      <p>Token address A:
        <input v-model="tokena"> 
      </p>
      <p>Amount A:
        <input v-model="amounta"> 
      </p>
      <p>Token address B:
        <input v-model="tokenb">
      </p>
      <p>Amount B:
        <input v-model="amountb"> 
      </p>
      <a href="#" @click="approve">Approve both</a>
      <a href="#" @click="add">Add liquidity</a>
    </div>
    <div v-else>Please, connect your wallet first</div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { ethers } from "ethers";
const routerAbi = [
  "function addLiquidity(address tokenA, address tokenB, uint amountADesired, uint amountBDesired, uint amountAMin, uint amountBMin, address to,        uint deadline    ) external returns (uint amountA, uint amountB, uint liquidity)"
];

const erc20Abi = [
  "function balanceOf(address owner) view returns (uint256)",
  "function approve(address spender, uint256 amount) public returns (bool)"
];
export default {
  name: "Add",
  data: function () {
     return {
       tokena: "0xa4Aa437E7b1F1ee49ad6CD291c9fdAE37707F5dA",
       tokenb: "0x7db03a15Aa2A1Cd87034FE2E9e08D2e422f51c9A",
       amounta: 0,
       amountb: 0,
     }
  },
  computed: {
    ...mapGetters("accounts", ["getChainName", "isUserConnected", "getActiveAccount", "getProviderEthers"]),
  },
  methods: {
    async add() {
      const Router = new ethers.Contract('0x7740B7a39AED690075E05a5335fcC7C2D6E04311', routerAbi,  this.getProviderEthers.getSigner())

      const trans = await Router.addLiquidity(
        this.tokena,
        this.tokenb,
        ethers.utils.parseEther(this.amounta),
        ethers.utils.parseEther(this.amountb),
        0,
        0,
        this.getActiveAccount,
        ethers.constants.MaxUint256
      );

      await trans.wait()
      console.log(trans)
      
    },
    async approve() {
      const Router = new ethers.Contract('0x7740B7a39AED690075E05a5335fcC7C2D6E04311', routerAbi,  this.getProviderEthers.getSigner())

      const Token1 = new ethers.Contract(this.tokena, erc20Abi, this.getProviderEthers.getSigner())
      const Token2 = new ethers.Contract(this.tokenb, erc20Abi, this.getProviderEthers.getSigner())

      const t1Approve = await Token1.approve(Router.address, ethers.constants.MaxUint256)
      const t2Approve = await Token2.approve(Router.address, ethers.constants.MaxUint256)

      await t1Approve.wait()
      await t2Approve.wait()

      
    }

  }
}
</script>
