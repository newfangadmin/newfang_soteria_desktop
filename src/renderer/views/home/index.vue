<template>
  <div>
    <folders />
    <files />
    <transition name="slide">
      <fdetails v-if="show" :details="details" class="detailsContainer" :contract="didContract" />
    </transition>
    <div :class="{ overlay: hasOverlay }" @click="closeDetails()"></div>
  </div>
</template>

<script>
import folders from '@/components/Folders'
import files from '@/components/Files'
import fdetails from '@/components/Details'
const ethers = require('ethers')

export default {
  components: {
    folders,
    files,
    fdetails
  },
  data () {
    return {
      curFolderName: '',
      curFolderId: '',
      show: false,
      hasOverlay: false,
      details: {},
      didContract: null
    }
  },
  computed: {
  },
  methods: {
    closeDetails () {
      this.show = false
      this.hasOverlay = false
    }
  },
  mounted () {
    this.$root.$on('showFileDetailsPane', (obj) => {
      this.show = true
      this.hasOverlay = true
      this.details = { ...obj }
    })

    this.$root.$on('closeDetailsPane', () => {
      this.closeDetails()
    })

    const abi = [
      {
        "inputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "constructor"
      },
      {
        "anonymous": false,
        "inputs": [
          {
            "indexed": false,
            "internalType": "bytes32",
            "name": "key",
            "type": "bytes32"
          },
          {
            "indexed": false,
            "internalType": "uint256",
            "name": "validity",
            "type": "uint256"
          }
        ],
        "name": "KeyHash",
        "type": "event"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "name": "accessSpecifier",
        "outputs": [
          {
            "internalType": "bytes32",
            "name": "encrypted_key",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "validity",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_new_owner",
            "type": "address"
          }
        ],
        "name": "changeFileOwner",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_new_owner",
            "type": "address"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "changeOwnerSigned",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "name": "changed",
        "outputs": [
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_id",
            "type": "bytes32"
          }
        ],
        "name": "createDID",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_id",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "createDIDSigned",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "n",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "k",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "file_size",
            "type": "uint256"
          },
          {
            "internalType": "string",
            "name": "ueb",
            "type": "string"
          }
        ],
        "name": "fileUpdate",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "n",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "k",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "file_size",
            "type": "uint256"
          },
          {
            "internalType": "string",
            "name": "ueb",
            "type": "string"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "fileUpdateSigned",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          }
        ],
        "name": "files",
        "outputs": [
          {
            "internalType": "uint256",
            "name": "n",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "k",
            "type": "uint256"
          },
          {
            "internalType": "uint256",
            "name": "file_size",
            "type": "uint256"
          },
          {
            "internalType": "string",
            "name": "ueb",
            "type": "string"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          }
        ],
        "name": "getAllUsers",
        "outputs": [
          {
            "internalType": "address[]",
            "name": "",
            "type": "address[]"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          }
        ],
        "name": "getKeyHash",
        "outputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "getKeyHashSigned",
        "outputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "payloadHash",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "getSigner",
        "outputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "payable": false,
        "stateMutability": "pure",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          }
        ],
        "name": "getTotalUsers",
        "outputs": [
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [],
        "name": "log",
        "outputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "name": "nonce",
        "outputs": [
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [],
        "name": "owner",
        "outputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          }
        ],
        "name": "owners",
        "outputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_user",
            "type": "address"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_key",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "_validity",
            "type": "uint256"
          }
        ],
        "name": "share",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_user",
            "type": "address"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_key",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "_validity",
            "type": "uint256"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "shareSigned",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_user",
            "type": "address"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_key",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "_validity",
            "type": "uint256"
          }
        ],
        "name": "updateACK",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "_file",
            "type": "bytes32"
          },
          {
            "internalType": "address",
            "name": "_user",
            "type": "address"
          },
          {
            "internalType": "bytes32",
            "name": "_access_type",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "_access_key",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "_validity",
            "type": "uint256"
          },
          {
            "internalType": "address",
            "name": "signer",
            "type": "address"
          },
          {
            "internalType": "uint8",
            "name": "v",
            "type": "uint8"
          },
          {
            "internalType": "bytes32",
            "name": "r",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "s",
            "type": "bytes32"
          }
        ],
        "name": "updateACKSigned",
        "outputs": [
          {
            "internalType": "bool",
            "name": "",
            "type": "bool"
          }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "",
            "type": "bytes32"
          },
          {
            "internalType": "uint256",
            "name": "",
            "type": "uint256"
          }
        ],
        "name": "userAccess",
        "outputs": [
          {
            "internalType": "address",
            "name": "",
            "type": "address"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      }
    ]
    const didContractAddress = '0xBedE189FC89124177876012f113fd2ecFBeBa15a'
    const providerAddress = 'https://testnet2.matic.network'
    const provider = new ethers.providers.JsonRpcProvider(providerAddress)
    this.didContract = new ethers.Contract(didContractAddress, abi, provider)
  }
}
</script>

<style lang="scss" scoped>
.overlay {
  z-index: 998;
  position: fixed;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
}

.fab-main-container {
  right: 5% !important;
  z-index: 2000 !important;
}

.detailsContainer {
  position: fixed;
  height: 100vh;
  width: 30%;
  top: 0;
  right: 0;
  background: #f2f2f2;
  z-index: 999;
  border-left: 1px solid #ddd;
}
</style>
