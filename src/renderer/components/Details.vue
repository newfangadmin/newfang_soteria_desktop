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
        <el-row :gutter="10">
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Actual Size</div>
              <div class="cardValue">{{ actualSize }}</div>
            </el-card>
          </el-col>
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Stored Size</div>
              <div class="cardValue">25.62MB</div>
            </el-card>
          </el-col>
          <el-col :span="8" class="infoCards">
            <el-card>
              <div class="cardHeader">Shared With</div>
              <div class="cardValue">4</div>
            </el-card>
          </el-col>
        </el-row>
        <el-row class="sharedWithArea">
          <el-col :span="24" class="sharedWithHeader">Shared With</el-col>
          <el-row class="sharedWithCards">
            <el-col :span="24" v-for="n in 4">
              <el-card class="sharedWithCard">
                <el-col :span="18" class="userDetails">
                  <div class="nickName">Joe Doe</div>
                  <div class="publicKey">0x10c87a669ea9707502cf5f763fc1fcac316a063c</div>
                </el-col>
                <el-col :span="6" class="revokeBtnContainer">
                  <el-button class="secondaryBtn">Revoke</el-button>
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

export default {
  components: {
  },
  data () {
    return {
      loading: false,
      fileName: '',
      addDate: '',
      actualSize: '',
      uri: '',
      storedSize: ''
    }
  },
  computed: {
  },
  methods: {
    closeDetails () {
      this.$root.$emit('closeDetailsPane')
    }
  },
  mounted () {
    console.log('hi1')
    const self = this
    this.$root.$on('showFileDetailsPane', ({id, name, size, uri}) => {
      console.log('hi2')
      console.log(id, name, size, uri)
      this.$db.find({_id: id}, function (err, docs) {
        if (!err) {
          // console.log(docs)
          self.fileName = docs[0].name
          self.addDate = docs[0].addDate
          self.actualSize = docs[0].size
          self.uri = docs[0].uri
          // console.log(self.fileName)
        }
      })
    })
  }
}
</script>

<style lang="scss" scoped>
.closeDetailsIcon {
  font-size: 20px;
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
  margin-bottom: 12px;
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