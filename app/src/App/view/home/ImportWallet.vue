<template>
  <div class="importWallet_container">
    <asset-back title style="padding-top:0" :backPath="backPath"></asset-back>
    <section class="content">
      <div class="words">
        <p>请输入您15位钱包助记词，用空格分隔！</p>
        <!-- <div class="seed-ui__word-group_zh">
          <one-box-one-word :box-num="15" v-model="seedStringInput"></one-box-one-word>
        </div>-->
        <div class="seed-ui__word-group_en">
          <textarea  v-model="seedStringInput" @keydown.enter="$event.preventDefault()" @input="inputHandle"></textarea>
        </div>
      </div>
      <el-form
        label-position="top"
        :rules="createRules"
        :model="createForm"
        ref="createForm"
        class="password"
      >
        <el-form-item label="请输入您的密码（8-16位字符）" prop="pwd">
          <el-input v-model="createForm.pwd" type="password" @input="inputHandle"></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="confirmPwd">
          <el-input v-model="createForm.confirmPwd" type="password" @input="inputHandle"></el-input>
        </el-form-item>
      </el-form>
      <div class="btn">
        <div @click="importWallet">导入</div>
      </div>
    </section>
  </div>
</template>

<script>
import AssetBack from "@/components/AssetBack.vue";
import OneBoxOneWord from "@/components/OneBoxOneWord.vue";
import {setChromeStorage,getChromeStorage} from '@/libs/chromeUtil.js'
import { createNamespacedHelpers } from 'vuex'
import { encrypt,decrypt } from "@/libs/crypto.js";
import walletAPI from "@/mixins/walletAPI.js";
import recover from "@/mixins/recover.js";
import importOrExchange from "@/mixins/importOrExchange.js";
const { mapState } = createNamespacedHelpers('Account')
export default {
  components: { AssetBack, OneBoxOneWord },
  mixins:[walletAPI,recover,importOrExchange],
  computed:{
    ...mapState(['currentMain']),
  },
  data() {
    let confirmPwdValidate = (rule, value, callback) => {
      if (value !== this.createForm.pwd) {
        callback(new Error("两次输入密码不一致"));
      } else {
        callback();
      }
    };
    return {
      seedStringInput: "",
      createForm: {
        pwd: "",
        confirmPwd: ""
      },
      createRules: {
        pwd: [
          { required: true, message: "请输入您的密码", trigger: "blur" },
          { min: 8, max: 16, message: "8 到 16位字符", trigger: "blur" }
        ],
        confirmPwd: [
          { required: true, message: "请输入您的确认密码", trigger: "blur" },
          { validator: confirmPwdValidate, trigger: "blur" }
        ]
      },
      extensionStatus:false,
      backPath:'/'
    };
  },
  methods: {
    inputHandle(){
      this.getAndSet('createForm',this.createForm)
      this.getAndSet('seedStringInput',this.seedStringInput)
    },
    importWallet(){
      if(!this.seedStringInput){
        this.$message.error('请输入助记词')
        return
      }
      this.$refs['createForm'].validate(valid=>{
        if(valid){
          this.saveSeed(this.seedStringInput, this.createForm.pwd).then(res=>{})
          // const walletObj = this.createHDWallet(this.seedStringInput)
          // this.newAccount('test')
          setTimeout(() => {
            this.$store.commit("Account/UPDATE_PASSWORD", this.createForm.pwd)
            this.$router.push({ name: 'WalletIndex' })
          }, 500)

        }
      })
    },
    // saveSeed (seedString, password) {
    //   const walletObj = this.createHDWallet(seedString)
    //   // 加密助记词 
    //   let ciphertext = encrypt(seedString, password)
    //   let obj = {
    //     ciphertext,
    //     password,
    //     // wallet : JSON.stringify(walletObj),
    //     // account : walletObj.accountMap[0],
    //     // isLogout:false
    //   }
    //   // 在AccountList中找出登出的钱包
    //   getChromeStorage("AccountList").then(res=>{
    //     console.log('=====res.AccountList==========')
    //     console.log(res.AccountList)
    //     console.log(res.AccountList.length)
    //     console.log(JSON.parse(res.AccountList[0]))
    //     // console.log(JSON.parse(res.AccountList[1]))
    //     if(res.AccountList){
    //       let account = {}
    //       for(let i=0; i<res.AccountList.length;i++){
    //         let pA = JSON.parse(res.AccountList[i])
    //         let mnemonic = decrypt(pA.ciphertext, pA.password);
    //         if(seedString == mnemonic){
    //           console.log('存在该钱包')
    //           console.log(pA)
    //           // account = {...pA}
    //           account = {...pA,...obj}
    //           break
    //         }
    //       }
    //       console.log(account)
    //       if(!account.name){
    //         alert('error：导入钱包找不到wallet')
    //         return
    //       }
    //       setChromeStorage("CreateingWallet", account).then(res=>{
    //         console.log('=====钱包账户存储成功=====')
    //         this.newAccount(account.name,'import')
    //       })
    //     }else{
    //       this.$message.error("无AccountList3");
    //     }
    //   })
    //   // this.newAccount('创世地址')
    //   return walletObj
    // },
  },
  mounted(){
    getChromeStorage("extensionStatus").then(res=>{
      console.log(res)
      if (res.extensionStatus == 'add'){
        this.extensionStatus = 'add';
        this.backPath = '/ImportOrCreate'
      }else if (res.extensionStatus == 'lock'){
        this.extensionStatus = 'lock';
        this.backPath = '/'
      }else{
        this.extensionStatus = '';
      }
    })
  }
};
</script>

<style lang='scss'>
.importWallet_container {
  .content {
    width: 344px;
    margin: 0 auto;
    margin-top: 24px;
    .words {
      margin-bottom: 22px;
      > p {
        font-size: 16px;
        color: #ffffff;
        margin-bottom: 7px;
      }
      > div {
        height: 120px;
        background: #ffffff;
        padding: 15px 30px 15px 30px;
        border-radius: 10px;
        box-shadow: 2px 2px 5px 3px #ffffff;
      }
      .seed-ui__word-group_zh {
        .word-box {
          height: 36px;
          width: 36px;
          margin-left: 21px;
          font-size: 26px;
          text-align: center;
          &:nth-child(5n + 1) {
            margin-left: 0;
          }
        }
      }
      .seed-ui__word-group_en {
        > textarea {
          width: 100%;
          height: 100%;
          display: block;
          border: none;
          outline: none;
          font-size: 16px;
          resize: none;
          color: #FF6A8B;
          font-family: MicrosoftYaHei;
        }
      }
    }
    .password {
      .el-form-item {
        margin-bottom: 15px;
      }
      .el-form-item__label {
        font-size: 16px;
        color: #ffffff;
        padding: 0;
        line-height: 30px;
        &::before {
          content: "";
        }
      }
      input {
        border-radius: 10px;
        height: 38px;
      }
    }
    .btn {
      margin: 0 auto;
      margin-top: 36px;
      width: 344px;
      height: 66px;
      background-image: url("../../../assets/images/loginBtn.png");
      background-size: 100% 100%;
      text-align: center;
      font-size: 16px;
      font-family: MicrosoftYaHei;
      font-weight: 400;
      // padding: 12px 0 18px;
      >div {
        &:hover{
          cursor: pointer;
        }
        width: 100%;
        height: 100%;
        padding-top: 10px;
        display: inline-block;
        color: rgba(255, 255, 255, 1) !important;
        // margin-top: 6px;
      }
    }
  }
}
</style>
