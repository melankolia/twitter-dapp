<template>
  <div class="flex flex-col h-screen">
    <header>
      <div class="flex flex-col justify-center items-center pt-20">
        <p class="text-5xl font-bold text-[#19a7f4]">Twitter Dapp</p>
        <button
          class="mt-5 px-3.5 py-2.5 rounded-full bg-[#19a7f4] text-white"
          @click="handleConnect"
        >
          <span v-if="!isWalletAvail">Connect Wallet</span>
          <span v-else>Disconnect</span>
        </button>
        <p class="text-[#19a7f4] my-2">CONNECTED: {{ wallet }}</p>
      </div>
    </header>
    <main>
      <div class="flex flex-col justify-center px-20">
        <textarea
          class="py-2 px-3 rounded-xl border border-dark-50"
          v-model="tweet"
          placeholder="What's Happening"
        />
        <button
          class="my-5 px-3.5 py-2 rounded-full bg-[#19a7f4] text-white self-start"
          @click="handleTweet"
        >
          <span v-if="loading">Loading ...</span>
          <span v-else>Tweet</span>
        </button>
      </div>
    </main>
    <div v-if="loadingTweet" class="flex flex-col mx-20 justify-center items-center">
      <p class="text-3xl font-bold text-[#19a7f4] animate-pulse">Loading Tweet's ...</p>
    </div>
    <div v-else class="overflow-scroll">
      <div
        v-for="(tweet, index) in tweets"
        class="flex flex-col mx-20 border border-dark-50 my-4"
        :key="index"
      >
        <div @click="() => handleLike(index, tweet)" class="flex flex-row">
          <img width="120" :src="imageLink" alt="Avatar" />
          <div class="flex flex-col justify-center px-4 truncate">
            <p class="text-lg font-bold text-[#19a7f4]">{{ tweet.author }}</p>
            <p class="text-lg text-black">{{ tweet.content }}</p>
            <div class="flex">
              <p v-if="tweet.loading" class="text-sm font-bold text-[#19a7f4]">Loading ...</p>
              <p
                v-else
                class="text-lg font-bold text-[#19a7f4] cursor-pointer duration-500 rounded-xl hover:bg-[#19a7f4] hover:text-white hover:px-2 active:opacity-70"
              >
                {{ tweet.likes || 0 }} Like's
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import detectEthereumProvider from '@metamask/detect-provider'
import contractABI from './assets/abi.json'
import Web3 from 'web3'

const imageLink = ref(
  'https://fiverr-res.cloudinary.com/images/t_main1,q_auto,f_auto,q_auto,f_auto/gigs/195538712/original/83171f4c57173b5ed97c0c8313d52d691108c2a5/create-a-pixel-art-8-bit-avatar-for-you.jpg'
)
const loading = ref(false)
const loadingTweet = ref(false)
const wallet = ref(null)
const twitterContract = ref({})
const tweet = ref(null)
const tweets = ref([])
const contractAddress = ref(import.meta.env.VITE_CONTRACT_ADDRESS)

const isWalletAvail = computed(() => {
  return wallet.value
})

const handleTweet = async () => {
  try {
    loading.value = true
    await twitterContract.value.methods
      .createTweet(tweet.value)
      .send({
        from: wallet.value
      })
      .then(function (result) {
        console.log({ result })
        tweet.value = null
        getAllData()
      })
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

const handleLike = async (_index, tweet) => {
  try {
    tweets.value[_index].loading = true
    await twitterContract.value.methods
      .likeTweet(tweet.author, tweet.id)
      .send({ from: wallet.value })

    tweets.value[_index].likes++
  } catch (error) {
    console.error(error)
  } finally {
    tweets.value[_index].loading = false
  }
}

const handleConnect = async () => {
  const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' }).catch((err) => {
    if (err.code === 4001) {
      // EIP-1193 userRejectedRequest error.
      // If this happens, the user rejected the connection request.
      console.log('Please connect to MetaMask.')
    } else {
      console.error(err)
    }
  })
  wallet.value = accounts[0]
}

const setupContract = async () => {
  let web3 = new Web3(window.ethereum)
  let localTwitterContract = new web3.eth.Contract(contractABI, contractAddress.value)
  twitterContract.value = localTwitterContract
}

const setupMetamask = async () => {
  const provider = await detectEthereumProvider()

  if (provider && provider === window.ethereum) {
    console.log('MetaMask is available!')
    startApp(provider) // Initialize your dapp with MetaMask.
  } else {
    console.log('Please install MetaMask!')
  }
}

const startApp = (provider) => {
  if (provider !== window.ethereum) {
    console.error('Do you have multiple wallets installed?')
  }
}

const getAllData = async (wallet) => {
  try {
    loadingTweet.value = true
    const tempTweets = await twitterContract.value.methods.getAllTweets(wallet).call()
    tempTweets.map((e) => {
      e.loading = false
    })
    tweets.value = [...tempTweets]
  } catch (error) {
    console.error(error)
  } finally {
    loadingTweet.value = false
  }
}

watch(wallet, (newWallet) => {
  newWallet && getAllData(newWallet)
})

onMounted(() => {
  setupMetamask()
  setupContract()
})
</script>

<!-- 
<style scoped>
</style> -->
