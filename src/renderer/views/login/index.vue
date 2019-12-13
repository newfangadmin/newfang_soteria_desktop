<template>
  <div class="viewContainer loading" v-loading="loading" element-loading-background="rgba(0, 0, 0, 0.8)">
    <div class="signupArea" v-if="newUser">
      <el-row class="step1" v-if="step==1">
        <el-col :span="12" :offset="6" class="step1">
          <h3>Welcome</h3>
          <img class="logo" src="~@/assets/horizontal_white.png" /><br />
          <el-button @click="generateMnemonic()" class="mneBtn">Generate Mnemonic</el-button>
        </el-col>
      </el-row>
      <el-row class="mnemonic" v-if="step==2">
        <el-col :span="12" :offset="6" class="step2">
          <el-row>
            <h3>12 word Mnemonic used to generate your wallet address</h3>
            <el-col :span="3" :offset="1" v-for="(m, index) in mnemonic" :key="index" class="mnes">{{ m }}</el-col><br/>
            <el-col :span="16" :offset="4" class="step2Btns">
              <el-button @click="copyMnemonic()">Copy Mnemonic</el-button>
              <el-button @click="confirmCopy()">Enter Password</el-button><br/>
              <el-button @click="step=1" class="secondaryBtn">Back</el-button>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
    <div class="loginArea" v-else>
      <el-row class="userCreated" v-if="userJustCreated">
        <el-col :span="12" :offset="6" class="step3">
          <h3>Account created successfully.</h3>
          <el-button @click="gotoLogin()">Proceed to Login</el-button>
        </el-col>
      </el-row>
      <el-row class="userLogin" v-else>
        <el-col :span="12" :offset="6" class="step3">
          <h3>Login</h3>
          <img class="logo" src="~@/assets/horizontal_white.png" /><br />
          <input type="file" ref="fileSelect" style="display: none" v-on:change="handleKeystoreSelect()">
          <el-button class="chooseKeystoreBtn" @click="chooseKeystore()">Choose Keystore</el-button>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import { ethers } from 'ethers'
const bip39 = require('bip39')
const { clipboard } = require('electron')
const fs = require('fs')
const os = require('os')
const storage = require('electron-json-storage')
var random = require('random-name')

export default {
  components: {
  },
  data () {
    return {
      loading: false,
      newUser: true,
      mnemonic: [],
      step: 1,
      mne: '',
      userJustCreated: false,
      keystore: ''
    }
  },
  methods: {
    generateMnemonic () {
      this.mne = bip39.generateMnemonic()
      this.mnemonic = this.mne.split(' ')
      this.step = 2
    },

    copyMnemonic () {
      clipboard.writeText(this.mne)
      this.$message({
        type: 'success',
        message: 'Mnemonic Copied to clipboard. Please copy the Mnemonic somewhere safe!',
        duration: 10000
      })
    },

    confirmCopy () {
      this.$confirm('I confirm that I have copied the Mnemonic somewhere safe. OK to continue or Cancel to copy again.', 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        this.enterPassword()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: 'Please copy the Mnemonic somewhere safe!'
        })
      })
    },

    enterPassword () {
      this.$prompt('Please input a password that will be used to login to Newfang.<br/>Password must have:<ul><li>At least one lower case alphabet</li><li>At least one upper case alphabet</li><li>At least one number</li><li>At least 8 characters long</li>', 'Choose Password', {
        confirmButtonText: 'Generate Keystore',
        cancelButtonText: 'Cancel',
        dangerouslyUseHTMLString: true,
        inputPattern: /^(((?=.*[a-z])(?=.*[A-Z]))|((?=.*[a-z])(?=.*[0-9]))|((?=.*[A-Z])(?=.*[0-9])))(?=.{8,})/,
        inputErrorMessage: 'Password not strong enough.',
        inputType: 'password'
      }).then(({ value }) => {
        this.generateKeystore(value)
      }).catch(() => {
      })
    },

    generateKeystore (password) {
      let mnemonicWallet = ethers.Wallet.fromMnemonic(this.mne)

      let encryptPromise = mnemonicWallet.encrypt(password, (progress) => {
        console.log('Encrypting: ' + parseInt(progress * 100) + '% complete')
      })

      const self = this
      encryptPromise.then(function (json) {
        const blob = new Blob([json], {type: 'text/plain'})
        const e = document.createEvent('MouseEvents')
        let a = document.createElement('a')
        a.download = 'newfang_keystore.json'
        a.href = window.URL.createObjectURL(blob)
        a.dataset.downloadurl = ['text/json', a.download, a.href].join(':')
        e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
        a.dispatchEvent(e)
        self.addUser(password)
      })
    },

    addUser (pass) {
      const self = this
      var newUser = {
        firstName: random.first(),
        lastName: random.last(),
        pass: pass,
        addDate: new Date(),
        storageCap: 1000000000,
        bandwidthCap: 1000000000,
        sUsage: 0,
        bUsage: 0
      }
      this.$udb.insert(newUser, function (err, newDoc) {
        if (!err) {
          var homeFolder = {
            name: 'home',
            uid: newDoc._id,
            addDate: new Date(),
            type: 'folder',
            parentId: 'null'
          }
          self.$db.insert(homeFolder, function (err, newDoc1) {
            if (!err) {
              self.newUser = false
              self.userJustCreated = true
            }
          })
        }
      })
    },

    gotoLogin () {
      this.userJustCreated = false
    },

    chooseKeystore () {
      this.$refs.fileSelect.click()
    },

    handleKeystoreSelect () {
      const file = this.$refs.fileSelect.files[0]
      if (file !== undefined) {
        fs.readFile(file.path, 'utf-8', (err, data) => {
          if (err) {
            alert('An error ocurred reading the file :' + err.message)
            return
          }
          this.keystore = data
          this.getPassword()
        })
      }
    },

    getPassword () {
      this.$prompt('Please enter your password', 'Enter Password', {
        confirmButtonText: 'Confirm',
        cancelButtonText: 'Cancel',
        inputType: 'password'
      }).then(({ value }) => {
        this.loading = true
        this.checkPassword(value)
      }).catch(() => {
      })
    },

    checkPassword (value) {
      const self = this
      this.$udb.find({pass: value}, function (err, docs) {
        if (!err) {
          if (docs.length === 0) {
            self.$message({
              type: 'error',
              message: 'Incorrect password. Please try again.'
            })
            self.getPassword()
          } else {
            ethers.Wallet.fromEncryptedJson(self.keystore, value).then(function (wallet) {
              storage.setDataPath(os.tmpdir())
              storage.set('wallet', wallet, function (err) {
                if (!err) {
                  localStorage.setItem('uid', docs[0]._id)
                  self.$db.find({ uid: docs[0]._id, type: 'folder', parentId: 'null' }, function (err, docs1) {
                    if (!err) {
                      self.loading = false
                      localStorage.setItem('curFolderId', docs1[0]._id)
                      localStorage.setItem('curFolderName', docs1[0].name)
                      localStorage.setItem('curParent', 'null')
                      self.$router.push({name: 'Home', params: {fid: docs1[0]._id, parentId: 'null'}})
                    }
                  })
                }
              })
            })
          }
        }
      })
    }
  },
  mounted () {
    const self = this
    this.$udb.find({}, function (err, docs) {
      if (!err) {
        if (docs.length === 0) {
          self.newUser = true
        } else {
          self.newUser = false
        }
      }
    })
  }
}
</script>

<style lang="scss" scoped>
  .viewContainer {
    height: 100vh;
    width: 100%;
    background: #222;
    display: table;
    color: #f2f2f2;
  }

  .loginArea {
    text-align: center;
    margin-top: 80px;
  }

  .step1, .step2 {
    text-align: center;
    margin-top: 80px;
  }

  .logo {
    width: 200px;
  }

  .mneBtn, .chooseKeystoreBtn {
    margin-top: 44px;
  }

  .step2Btns {
    margin-top: 40px;
  }

  .mnes {
    font-size: 16px;
    margin-top: 40px;
    margin-bottom: 8px;
    text-align: center;
    border: 1px solid #f2f2f2;
    border-radius: 6px;
    padding-top: 1px;
    padding-bottom: 4px;
  }
</style>
