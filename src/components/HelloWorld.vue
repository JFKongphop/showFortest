

<script setup>
// dp contract in solidity
import { ref, onMounted, provide } from 'vue'
import { ethers, utils } from "ethers"
import abi from "../assets/abi.json"

defineProps({
  msg: String
})

  const input = ref("");
  const currentAddress = ref("");
  const currentNetwork = ref("");
  let provider = {};
  let signer = {};
  const contractAddress = "0x4B0848114979e047A1264f986e4B36429c0C81bC"
  
  async function transfer(){
    // const txCount = await provider.getTransactionCount(signer.getaAddress()); // get latest tx
    const rawTX = {
      //nouce : utils.hexlify(),
      to : "0xd73F821fcA522Cbb672F8354d25470DBf4948c9C",
      value : utils.parseEther(input.value),
    }

    const tx = await signer.sendTransaction(rawTX);
    console.log(tx.hash);
    await tx.wait();

    console.log("transfer");
    input.value = 0;
  }
// call function in smart contract
async function callGreet(){
  const ABI = ["function greet() public view returns (string)"];
  const iface = new utils.Interface(ABI);
  const encodeData = iface.encodeFunctionData("greet")
  
  // call contract
  const response = await provider.call({
    to: contractAddress,
    data: encodeData
  })
  console.log("call greet");
  console.log(utils.toUtf8String(response));
}

// call function in smart contract 
// function set 
async function callSetGreeting(){
  const ABI = ["function setGreeting(string _greeting)"];
  const iface = new utils.Interface(ABI);
  const encodeData = iface.encodeFunctionData("setGreeting", ["Hello World"]);

  // set is use signer not provider
  const tx = await signer.sendTransaction({
    to: contractAddress,
    data: encodeData
  })

  console.log(tx.hash);
  await tx.wait();
}

// call greet by json that get out file amd set in asset
async function callGreetWithContract(){
  const contract = new ethers.Contract(
    contractAddress,
    abi, 
    provider
  )
  const response = contract.greet();
  console.log(response);
}

// call and run transaction
async function callSetGreetWithContract(){
  // can set contract in onMounted and set to global variable for use 
  const contract = new ethers.Contract(
    contractAddress,
    abi, 
    provider // can set provider or signer
    // if use provider must set connect after contract.connect(signer)
    // if use signer not call many time connect(signer)
  ) 
  const tx = await contract.connect(signer).setGreeting("Kongphop_King");
  console.log(tx.hash);

  await tx.wait();
}

// use when conponent is loading
onMounted(async()=>{
  // provider field
  // use when connect metamask
  provider = new ethers.providers.Web3Provider(window.ethereum)

  // use when not connect metamask
  // danger
  // provider = new ethers.providers.JsonRpcProvider("")
  await provider.send("eth_requestAccounts", []);
  signer = provider.getSigner(); // account who connect

  // in metamask doc
  // show current address
  currentAddress.value = await signer.getAddress();

  // show chain
  const network = ref("");
  network.value = await provider.getNetwork();
  currentNetwork.value = await network._rawValue.name
  console.log(currentNetwork);

  // show switch account page is reload
  window.ethereum.on('accountsChanged', (accounts) => {
    window.location.reload();
  });

  // show when switch network page is reload
  window.ethereum.on('chainChanged', (chainId) => {
    window.location.reload();
  });

  // error will edit soon but not important

  // show logs in transaction in this contract after when use 
  const filter = {
    address: "0x4B0848114979e047A1264f986e4B36429c0C81bC",
    topics: [
        utils.id("SetGreeting(address string)"),
    ], 
    fromBlock: 0
  }
  const logs = await provider.getLogs(filter)
  console.log(logs);

  const contract =  new ethers.Contract(
    contractAddress,
    abi,
    signer
  )

  // const eventFilter =  contract.filter.SetGreeting()
  // const events = await contract.queryFilter(eventFilter)
  // events.forEach((event)=>{
  //   console.log(event.args.from, event.args.msg);
  // })

  // // when have is use this auto
  // contract.on("SetGreeting", (from, msg)=>{
  //   console.log(from, msg);
  // })


})

</script>

<template>
    <h3>{{currentAddress}}</h3>
    <h3>{{currentNetwork}}</h3>
    <input v-model="input"/>
    <button @click="transfer">Transfer</button>
    <br>
    <button @click="callGreet">Call Greet</button>
    <br>
    <button @click="callSetGreeting">Call Set Greet</button>
    <br>
    <button @click="callGreetWithContract">Call greet with contract</button>
    <br>
    <button @click="callSetGreetWithContract">Call set greeting with contract</button>

</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>

    // build transaction 
    // const rawTX = {
    //   from : signer.getAddress(),
    //   to : "0xd73F821fcA522Cbb672F8354d25470DBf4948c9C",
    //   value : utils.parseEther(input.value),
    //   gasLimit : utils.hexlify(30000),
    //   gasPrice : utils.hexlify(parseInt(await provider.getGasPrice()))
    // }

  // // block and network
  // console.log(await provider.getNetwork())
  // console.log(parseInt(await provider.getGasPrice()))
  // console.log(parseInt(await provider.getBlockNumber()))
  // // block detail
  // console.log(await provider.getBlock(await provider.getBlockNumber()))

  // let balance = await provider.getBalance("ether.eth")
  // let realNum = ethers.utils.formatEther(balance);
  // console.log(realNum);

  // // signer field
  // const signer = provider.getSigner();
  // // console.log(signer);

  // // get address login
  // console.log(await signer.getAddress());

  // // utilities
  // console.log(utils.parseEther("1.0")) // to ether
  // console.log(utils.formatEther("1000000000000000000")) // to wei
  // console.log(utils.formatUnits("1000000000000000000", "gwei")) // can set add uint
  // console.log(utils.hexlify(12345));// to binary 16
  // console.log(utils.hexlify(utils.toUtf8Bytes("JFKongphop")));
  // console.log(utils.toUtf8String("0x6b6f6e6770686f70206b696e6770657468"));**/