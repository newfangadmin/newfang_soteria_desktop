<template>
  <div v-loading="loading" element-loading-background="rgba(255, 255, 255, 0.7)">
    <el-row>
      <i class="el-icon-close closeDetailsIcon" @click="closeDetails()"></i>
    </el-row>
    <el-row>
      <el-col :span="20" :offset="2" class="fileInfo">
        <el-row>
          <el-col :span="24" class="fileName info">{{ fileName }}</el-col>
          <el-col :span="24" class="addDate info">{{ addDate }}</el-col>
        </el-row>
        <el-row style="text-align: center; margin: 10px;">
          <el-button class="smBtn copyDidBtn" @click="handleCopyDid">Copy File DID</el-button>
        </el-row>
        <el-row :gutter="10">
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Actual Size</div>
              <div class="cardValue">{{ actualSize | sizeFilter }}</div>
            </el-card>
          </el-col>
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Stored Size</div>
              <div class="cardValue">{{ storedSize | sizeFilter }}</div>
            </el-card>
          </el-col>
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Shared With</div>
              <div class="cardValue">{{ shareSize }}</div>
            </el-card>
          </el-col>
        </el-row>
        <el-row class="sharedWithArea">
          <el-col :span="24" class="sharedWithHeader">Shared With</el-col>
          <el-row class="sharedWithCards">
            <el-col :span="24" v-for="(r, index) in shareList" :key="index">
              <el-card class="sharedWithCard">
                <el-col :span="18" class="userDetails">
                  <div class="nickName">Joe Doe</div>
                  <div class="publicKey">{{ r }}</div>
                </el-col>
                <el-col :span="6" class="revokeBtnContainer">
                  <el-button class="secondaryBtn" @click="handleRevoke(r, index)">Revoke</el-button>
                </el-col>
              </el-card>
            </el-col>
          </el-row>
        </el-row>
      </el-col>
    </el-row>
  </div>
</template>

<script>
const { clipboard } = require('electron')
const ethers = require('ethers')
const { Utils } = require('newfang_node')

export default {
  components: {
  },
  props: {
    contract: {
      type: Object
    }
  },
  data () {
    const details = this.$attrs.details
    const k = details.uri.split(':')[4]
    const n = details.uri.split(':')[5]
    const storedSize = (details.size) * (n / k)
    return {
      loading: true,
      fileName: details.name,
      addDate: '',
      actualSize: details.size,
      uri: details.uri,
      did: '',
      storedSize: storedSize,
      id: details.id,
      shareList: null,
      shareSize: 0,
      pubKeys: null,
      wallet: null,
      privateKey: ''
    }
  },
  filters: {
    sizeFilter (value) {
      return (value / 1000000).toFixed(2) + 'MB'
    }
  },
  computed: {
  },
  methods: {
    closeDetails () {
      this.$root.$emit('closeDetailsPane')
    },

    showMsgBox (type, message) {
      this.$message({
        type: type,
        message: message,
        duration: 5000
      })
    },

    handleCopyDid () {
      clipboard.writeText(this.did)
      this.$message({
        type: 'success',
        message: 'File DID: ' + this.did + ' copied to clipboard',
        duration: 6000
      })
    },

    handleRevoke (address, i) {
      this.loading = true
      const pubKey = this.pubKeys[i]
      console.log(pubKey)
      const did = this.did
      const wallet = this.wallet
      const util = new Utils({ did, convergence: "asda" }, {
        blockchain: {
          provider: ethers,
          wallet
        }
      })
      util.setIdentity(this.privateKey)
      util.on('response', (data) => {
        console.log(data)
        this.showMsgBox('success', 'File access to Public Key revoked.')
        this.loading = false
        console.log({ data })
      })
      util.on('error', (error) => {
        console.log({ error })
      })
      util.revoke(address)
    }
  },
  mounted () {
    const self = this
    this.$db.find({_id: this.id}, async function (err, docs) {
      if (!err) {
        self.pubKeys = docs[0].pubKeys
        self.addDate = docs[0].addDate
        self.did = docs[0].did
        const fileId = self.did.split(':')[2]
        self.shareList = await self.contract.getAllUsers(fileId, ethers.utils.formatBytes32String('read'))
        console.log(self.shareList)
        self.shareSize = self.shareList.length
        self.loading = false
      }
    })

    this.uid = localStorage.getItem('uid')
    this.$udb.find({_id: self.uid}, function (err, docs) {
      if (!err) {
        self.privateKey = docs[0].wallet.signingKey.privateKey
        self.wallet = new ethers.Wallet(self.privateKey)
      }
    })
  }
}
</script>

<style lang="scss" scoped>
.closeDetailsIcon {
  font-size: 16px;
  text-align: right;
  margin: 10px;
  color: #aaaaaa;
  font-weight: bold;
  cursor: pointer;
}

.infoHeader {
  text-align: center;
  font-size: 14px;
  margin-bottom: 10px;
  font-weight: bold;
  color: #aaaaaa;
}

.info {
  text-align: center;
}

.fileName {
  font-size: 18px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  height: 30px;
  line-height: 30px;
  font-weight: 600;
  color: #aaaaaa;
}

.addDate{
  font-size: 10px;
  color: #aaaaaa;
}

.infoCards {
  text-align: center;
}

.cardHeader {
  font-size: 12px;
  font-weight: 600;
  color: #aaaaaa;
  margin-bottom: 10px;
}

.cardValue {
  font-size: 18px;
  font-weight: 600;
}

.sharedWithArea {
  margin-top: 30px;
  text-align: center;
}

.sharedWithHeader {
  font-weight: 600;
  color: #aaaaaa;
  margin-bottom: 12px;
  height: 24px;
}

.sharedWithCards {
  overflow: auto;
  width: 100%;
  height: 66vh;
}

.sharedWithCard {
  margin-bottom: 10px;
}

.userDetails {
  text-align: left;
  height: 60px;
}

.nickName {
  font-size: 18px;
  font-weight: 600;
}

.publicKey {
  font-size: 12px;
  color: #aaaaaa;
}

.revokeBtnContainer {
  text-align: right;
}
</style>