<template>
  <div class="CreateNewToken">
    <el-row :gutter="22">
      <!-- left bar -->
      <el-col :span="16">
        <el-row :gutter="20">
          <el-col :span="8"><i>*</i> Receiver Name</el-col>
          <el-col :span="16"><el-input v-model="rTokenInfo.RName"></el-input></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8"><i>*</i> Receiver ID</el-col>
          <el-col :span="16"
            ><div class="rid">{{rTokenInfo.RId}}</div></el-input
          ></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8"><i>*</i> Receiver IP</el-col>
          <el-col :span="16"><el-input v-model="rTokenInfo.RIP" class="ip"></el-input> <el-button @click="getIp">Get IP</el-button></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8"><i>*</i> Receiver Port</el-col>
          <el-col :span="16"><el-input v-model="rTokenInfo.RPort"></el-input></el-col>
        </el-row>
        <el-row :gutter="20" class="ReceiverTime">
          <el-col :span="8"><i>*</i> Receiver Time Span</el-col>
          <el-col :span="16">
            <el-row :gutter="20">
              <!-- Years -->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidYear">
                  <el-option
                    v-for="(item,index) in years"
                    :key="item"
                    :label="item-1"
                    :value="item-1"
                  >
                  </el-option>
                </el-select>
                <span>Y</span>
              </el-col>
              <!-- Month -->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidMonth">
                  <el-option
                    v-for="item in months"
                    :key="item"
                    :label="item-1"
                    :value="item-1"
                  >
                  </el-option>
                </el-select>
                <span>M</span>
              </el-col>
              <!-- Days-->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidDay">
                  <el-option
                    v-for="item in days"
                    :key="item"
                    :label="item-1"
                    :value="item-1"
                  >
                  </el-option>
                </el-select>
                <span>D</span>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <!-- Hours -->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidHour">
                  <el-option
                    v-for="item in hours"
                    :key="item"
                    :label="item-1"
                    :value="item-1"
                  >
                  </el-option>
                </el-select>
                <span>H</span>
              </el-col>
              <!-- Minuts -->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidMinute">
                  <el-option
                    v-for="item in mins"
                    :key="item"
                    :label="item-1"
                    :value="item-1"
                  >
                  </el-option>
                </el-select>
                <span>M</span>
              </el-col>
              <!-- Seconds -->
              <el-col :span="8">
                <el-select v-model="rTokenInfo.RValidSecond">
                  <el-option
                    v-for="item in seconds"
                    :key="item"
                    :label="item"
                    :value="item"
                  >
                  </el-option>
                </el-select>
                <span>S</span>
              </el-col>
            </el-row>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8">Receiver Email</el-col>
          <el-col :span="16"><el-input v-model="receiverEmail"></el-input></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8">Transceiver Name</el-col>
          <el-col :span="16"><el-input v-model="transceiverName"></el-input></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8">Transceiver PID</el-col>
          <el-col :span="16"><el-input v-model="TransceiverPID"></el-input></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="8">Transceiver Email</el-col>
          <el-col :span="16"><el-input v-model="transceiverEmail"></el-input></el-col>
        </el-row>
        <el-row :gutter="20">
          <el-button class="createInfo" @click="createToken"> Create</el-button>
        </el-row>
      </el-col>
      <!-- right bar-->
      <el-col :span="8" class="rightBar">
        <div class="qrcode">
          <img :src="'data:image/png;base64,' + qrcodeUrl" alt="" v-if="qrcodeUrl!=''">
        </div>
        <el-button class="createTabBtn" v-if="qrcodeUrl!=''">
          <a :href="'data:image/png;base64,' + qrcodeUrl" download="Token QRCode">Save as</a></el-button>
        <el-button class="createTabBtn" v-else>Save as</el-button>
        <div class="copy">{{tokenInfo.Token}}</div>
        <el-button class="CopyBtn" 
          v-clipboard:copy="tokenInfo.Token"
          v-clipboard:success="onCopy"
          v-clipboard:error="onError"> Copy
        </el-button>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import { mapState } from 'vuex';
import Tool from '../../js/utils';
export default {
  data () {
    return {
      rTokenInfo: {},
      receiverName: '',
      years: 100,
      months: 12,
      days: 31,
      hours: 24,
      mins: 60,
      seconds: 60,
      message: '',
      receiverEmail: '',
      transceiverName: '',
      TransceiverPID: '',
      transceiverEmail: '',
      tokenInfo: {},
      qrcodeUrl: ''
    }
  },
  created () {
    this.getStatus()
  },
  computed: mapState({
    //映射
    State: state => state
  }),
  methods: {
    getStatus () {
      this.State.websocketStore.send({
        isOnce: true,
        key: 'getCurrentRInfo',
        data: {
          "OperationType": 601,
          "ContentType": '',
          "CategoryId": 2152867072,
          "ClientRequestId": Tool.createUUID(),
          "Data": ""
        },
        success: (res) => {
          if (res.ErrorMessage === 'Success') {
            this.rTokenInfo = JSON.parse(res.ReturnValue)
            console.log(JSON.parse(res.ReturnValue))
          } else {
            this.$message({
              type: 'warning',
              message: res.ErrorMessage
            })
          }
        }
      })
    },
    createToken () {
      const params = {
        RName: this.rTokenInfo.RName,
        RIP: this.rTokenInfo.RIP,
        RPort: this.rTokenInfo.RPort,
        RYear: this.rTokenInfo.RValidYear,
        RMonth: this.rTokenInfo.RValidMonth,
        RDay: this.rTokenInfo.RValidDay,
        RHour: this.rTokenInfo.RValidHour,
        RMinute: this.rTokenInfo.RValidMinute,
        RSecond: this.rTokenInfo.RValidSecond,
        REmail: this.receiverEmail,
        TName: this.transceiverName,
        TIdHex: this.TransceiverPID,
        TEmail: this.transceiverEmail
      }
      this.State.websocketStore.send({
        isOnce: true,
        key: 'getCurrentRInfo',
        data: {
          "OperationType": 300,
          "ContentType": '',
          "CategoryId": 2152867072,
          "ClientRequestId": Tool.createUUID(),
          "Data": JSON.stringify(params)
        },
        success: (res) => {
          if (res.ErrorMessage === 'Success') {
            this.tokenInfo = JSON.parse(res.ReturnValue)
            this.qrcodeUrl = this.tokenInfo.QRBase64String
          } else {
            this.$message({
              type: 'warning',
              message: res.ErrorMessage
            })
          }
        }
      })
    },
    getIp () {
      this.State.websocketStore.send({
        isOnce: true,
        key: 'getIp',
        data: {
          "OperationType": 600,
          "ContentType": '',
          "CategoryId": 2152867072,
          "ClientRequestId": Tool.createUUID(),
          "Data": ""
        },
        success: (res) => {
          if (res.ErrorMessage === 'Success') {
            console.log(res.ReturnValue)
            this.rTokenInfo.RIP = res.ReturnValue;
          } else {
            this.$message({
              type: 'warning',
              message: res.ErrorMessage
            })
          }
        }
      })
    },
    onCopy () {
      this.$message.success('copy success');
    },
    onError () {
      this.$message.success('copy failure');
    },
  }
}
</script> 
<style lang="less" scoped>
.CreateNewToken {
  /deep/.el-input {
    input {
      height: 32px;
    }
  }
  i {
    color: #ff7979;
    margin-right: 5px;
  }
  .ip {
    width: 212px;
    margin-right: 10px;
  }
  .el-col-8 {
    padding-left: 10px;
    text-align: left;
  }
  .el-col-16 {
    padding-left: 0px;
    margin-bottom: 10px;
    line-height: 32px;
  }
  .el-select {
    width: 62px;
  }
  .ReceiverTime {
    .el-row {
      line-height: 42px;
    }
    span {
      margin-left: 10px;
    }
  }
  .rightBar {
    border-left: 2px solid #444;
    height: 464px;
    .qrcode {
      width: 197px;
      height: 197px;
      background: #252525;
      border: 1px dashed #444444;
      margin: 0 auto;
    }
    .copy {
      width: 197px;
      height: 128px;
      background: #444444;
      border-radius: 4px;
      margin: 0 auto;
      word-break: break-all;
    }
  }
  /deep/.el-button {
    background: #252525;
    border-radius: 4px;
    border: 1px solid #666666;
    color: #fff;
  }
  .createTabBtn {
    margin: 12px 0px 27px 117px;
    a {
      color: #fff;
    }
  }
  .CopyBtn {
    margin: 10px 0px 0px 135px;
  }
  .createInfo {
    border: none;
    float: right;
    margin-right: 10px;
    color: #fff;
    background: #33ab4f;
  }
}
</style>


// WEBPACK FOOTER //
// src/components/children/CreateNewToken.vue