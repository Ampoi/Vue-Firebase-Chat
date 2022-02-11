<template>
  <!--オーバーフロー(設定)-->
  <div id="background" :class="{ hideSettings: hideSetting }">
      <div id="box">
          <button
              id="close"
              v-on:click="close"
          >
              <i class="bi bi-x"></i>
          </button>
          <h1>設定</h1>
          <h2>名前の変更</h2>
          <input
              type="text"
              v-model="newName"
              :placeholder="name(myId)"
              id="nameInput"
          >
          <button
              class="btn inbtn"
              id="kettei"
              v-on:click="changeName"
          >名前を変える</button>
      </div>
  </div>
  <!--アプリ画面-->
  <div>
    <div id="sidebar">
      <button class="sb-button" v-on:click="showSettings">
        <i class="bi bi-gear-fill"></i>
      </button>
    </div>
    <main>
      <div id="messages">
        <Message
          v-for="message in messages"
          :key="message.key"
          :from="name(message.from)"
          :id="message.from"
          :content="markDown(message.content)"
          :when="time(message.when)"
        />
      </div>
      <div>
        <div id="messageBtns">
          <button class="messageBtn able" @click="clickThis('uploadImage')">
            <i class="bi bi-cloud-upload-fill"></i>
          </button>
          <input
            id="uploadImage"
            type="file"
            style="display: none;"
            @change="ImageIn"
            accept="image/png image/jpeg image/gif"
          />
          <button class="messageBtn" :class="{ able: canAddImage }" @click="addImage">
            <i class="bi bi-cloud-plus-fill"></i>
          </button>
        </div>
        <div id="messageInDiv">
          <textarea
            type="text"
            id="messageIn"
            :placeholder="name(myId) + `としてメッセージを送信`"
            @keydown.enter.ctrl="send"
          ></textarea>
        </div>
        <button v-on:click="send" class="btn" id="send">
          <i class="bi bi-telegram"></i> 送信
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import { getFirestore, collection, addDoc, onSnapshot, query, doc, getDoc, getDocs, setDoc, orderBy, limit } from "firebase/firestore";
import { getAuth, onAuthStateChanged } from "firebase/auth";
import { getStorage, ref, uploadBytes, getDownloadURL } from 'firebase/storage';

import Push from 'push.js';
import { marked } from 'marked';

import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import App from '../views/App.vue'

import Message from '../components/message.vue'

//router関連
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/',
    name: 'App',
    component: App
  }
]
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

const db = getFirestore();

export default {
  name: 'Home',
  methods: {
    //送信関数
    async send(){
      const messageIn = document.getElementById("messageIn");
      const sendText = messageIn.value;

      //送信するメッセージが入力されているときにのみ送信
      if (sendText != ""){
        if (this.myId != ""){
          //スパムによる送信回数
          if(Date.now() - this.lastSendTime <= 1000){
            console.log(this.iyagaraseyamete)
            this.iyagaraseyamete += 1
          }
          //送信
          if (this.iyagaraseyamete < 3){
            this.sendCore(sendText, messageIn)
          }else{
            if (this.lastSendTime <= 20000){
              this.iyagaraseyamete = 0
              this.sendText(sendText, messageIn)
            }else{
              alert(`
連続送信はサーバーに負荷をかける原因になります！
やめてください！
z(スパム送信後は20秒間のクールダウンが入ります)
              `)
            }
          }
        }else{
          alert("すみませんが、アカウント情報があなたの端末では認識されていません。再度ログインしてみてください。")
        }
      }
    },

    //送信本体の関数
    async sendCore(sendText, messageIn){
      const nowChat = await query(collection(db, "teams", "main", "chats"));
      try {
        addDoc(nowChat, {
          from: this.myId,
          content: sendText,
          when: Date.now()
        });
        messageIn.value = ""
        this.lastSendTime = Date.now()
      } catch (e) {
        console.error("Error adding document: ", e);
      }
    },

    //accountIDから名前取得
    name(id){
      const thisName = this.usersName[id];
      if (thisName) {
        return thisName
      }else{
        return "unknown"
      }
    },

    //タイムスタンプから時刻変換
    time(when){
      const timestamp = new Date(when);
      const year = timestamp.getFullYear();
      const mon = timestamp.getMonth() + 1;
      const day = timestamp.getDate();
      const hour = `0${timestamp.getHours()}`.slice(-2);
      const min = `0${timestamp.getMinutes()}`.slice(-2);
      return `${year}/${mon}/${day} ${hour}:${min}`
    },

    //設定の表示
    showSettings(){
      this.hideSetting = false;
    },

    //設定閉じる
    close(){
      this.hideSetting = true;
      if (this.settingChanged == true){
        this.settingChanged = false;
        location.reload();
      }
    },

    //名前の変更
    async changeName() {
      if (this.newName != ""){
        await setDoc(doc(db, "users", this.myId), {
          name: this.newName
        });
        alert("名前が変更されました！");
        this.settingChanged = true;
      }
    },

    //メッセージのマークダウン化
    markDown(text) {
      const apapa = marked.parse(text);
      return apapa
    },

    //要素のクリック
    clickThis(element) {
      document.getElementById(element).click();
    },

    //画像のアップロード
    ImageIn() {
      const imageIn = document.getElementById("uploadImage");
      const uploadImage = imageIn.files[0];
      const kb = 1024;
      const imageSize = Math.trunc(uploadImage.size / kb);
      if (imageSize < 10000){
        const newImage = uploadImage.name;
        const storage = getStorage();
        const storageRef = ref(storage, `sendImages/${newImage}`);
        uploadBytes(storageRef, uploadImage).then(() => {
          getDownloadURL(storageRef)
          .then((url) => {
            this.addImageURL = `![](${url})`
            this.canAddImage = true
          })
          .catch((error) => {
            console.log(error);
          });
        });
      }else{
        alert("ファイル重いですよー\nもっと軽くしてください")
      }
    },

    //画像の挿入
    addImage() {
      //テキストエリアと挿入する文字列を取得
      var area = document.getElementById('messageIn');
      //カーソルの位置を基準に前後を分割して、その間に文字列を挿入
      area.value = area.value.substr(0, area.selectionStart)
          + this.addImageURL
          + area.value.substr(area.selectionStart);

      this.addImageURL = ""
      this.canAddImage = false
    }
  },
  mounted: async function(){
    //Authencation
    const auth = getAuth();

    onAuthStateChanged(auth, (user) => {
      if (user) {
        // User is signed in, see docs for a list of available properties
        const uid = user.uid;
        this.myId = uid;

        const usersRef = doc(db, "users", uid);
        const usersSnap = getDoc(usersRef);

        usersSnap.then(docSnap => {
          if (docSnap.exists()) {
            //すでにアカウント登録済み
          } else {
            //新規の人
            const newName = prompt("初めまして！KSChatで使用する名前を入力してください！")
            setDoc(doc(db, "users", this.myId), {
              name: newName
            }); 
          }
        })
      } else {
        // User is signed out
        this.$router.push({name: "Home"});
      }
    });

    //ユーザー情報の保存
    const querySnapshot = await getDocs(collection(db, "users"));
    querySnapshot.forEach((doc) => {
      // doc.data() is never undefined for query doc snapshots
      const id = doc.id;
      const name = doc.data().name;
      this.usersName[id] = name;
    });

    //リアルタイムで取得
    const nowChatRef = collection(db, "teams", "main", "chats");    
    onSnapshot(query(nowChatRef, orderBy("when", "desc"), limit(20)), async(querySnapshot) => {
      const newMessages = [];
      querySnapshot.forEach((doc) => {
        const newMessage = doc.data();
        newMessages.unshift(newMessage);
      });
      this.messages = newMessages;
      if (this.firstTime == false) {
        const apapa = await this.messages;
        console.log(`console${apapa}`)
        //スクロール位置を下げる
        const messages = document.getElementById("messages")
        if (messages.scrollHeight <= messages.scrollTop + messages.offsetHeight + 10) {
          this.$nextTick(function() {
            messages.scrollTop = messages.scrollHeight;
          });
        }
        //通知の表示
        const newMessage = this.messages[newMessages.length - 1];
        Push.create(`KSChat：${this.name(newMessage.from)}`, {
            body: newMessage.content,
            timeout: 4000,
            onClick: function () {
                window.focus();
                this.close();
            }
        });
      }else{
        this.firstTime = false;
      }
    })
  },
  data() {
    return {
      messages: [],
      myId: "",
      apapap: `
      aaa
      aaa`,
      usersName: {},
      hideSetting: true,
      settingChanged: false,
      firstTime: true,
      addImageURL: "",
      canAddImage: false,
      iyagaraseyamete: 0,
      lastSendTime: 0
    }
  },
  components: {
    Message
  },
  router
}
</script>

<style>
@import '/assets/cooldesign.css';

/*設定の部分*/
#background {
    background: rgba(0, 0, 0, 0.651);
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    width: 100%;
    transition: 0.5s;
}
.hideSettings {
    display: none;
}
#box {
    background-color: white;
    width: 760px;
    margin: 100px auto 0 auto;
    border-radius: 15px;
    padding: 20px;
}
#box h1 {
  font-size: 36px;
  font-weight: 500;
}
#close {
    font-size: 50px;
    float: right;
    padding: 0px;
    margin: 0px;
    background: none;
    border: none;
}
#kettei{
    margin-left: -121px;
}
#nameInput {
    border: none;
    background: rgb(233, 233, 233);
    border-radius: 10px;
    font-size: 18px;
    padding: 12px 130px 12px 12px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

/*アプリ画面*/
body {
  margin: 0px;
  overflow: none;
}
#sidebar {
  float: left;
  width: 50px;
  height: 100vh;
  background: rgb(233, 233, 233);
}
.sb-button {
  background: none;
  border: none;
  color: rgb(56, 165, 255);
  font-size: 34px;
  padding: 20px 0 0 8px;
}
main {
  width: calc(100% - 70px);
  height: calc(100vh - 20px);
  padding: 10px;
  float: left;
}
#messages {
  height: calc(100% - 134px);
  overflow: hidden;
  overflow-y: scroll;
}
#messageInDiv {
  border: none;
  background: rgb(233, 233, 233);
  border-radius: 10px;
  font-size: 18px;
  padding: 12px 80px 12px 12px;
  width: calc(100% - 92px);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  resize: none;
  height: 60px;
  float:left;
}
#messageBtns {
  background: rgb(233,233,233);
  margin: 0 0 10px 0;
  padding: 5px;
  border-radius: 10px;
  overflow: hidden;
  overflow-x: scroll;
}
.messageBtn {
  background: rgba(255, 0, 0, 0);
  color: #818fae;
  border: none;
  font-size: 20px;
  margin: 0 5px;
}
#messageIn {
  border: none;
  background: none;
  font-size: 18px;
  width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  resize: none;
  height: 60px;
  padding: 0px;
}
#send {
  padding: 5px;
  font-size: 18px;
  margin-left: -75px;
  margin-top: 25px;
}
.able {
  color: #282c34;
}

@media screen and (max-width: 840px) {
	/*スマホ(横幅が~480px)用のCSSをここに書く*/
  #box h1 {
    font-size: 30px;
    font-weight: 500;
  }
  #box h2 {
    font-size: 20px;
  }
  #box {
    width:calc(100% - 60px);
  }
}

@media screen and (max-width: 460px) {
	/*スマホ(横幅が~480px)用のCSSをここに書く*/
  #box input {
    width:calc(100% - 140px);
  }
}

@media (prefers-color-scheme: dark) {
  body {
    background-color: #282c34;
  }
  #sidebar, #messageInDiv {
    background-color: #21252b;
  }
  #messageIn {
    color: white;
  }
  #send {
    background-color: #2f343c;
  }
  #send:hover {
    background-color: rgb(56, 165, 255);
    color: #2f343c;
  }
  #close{
    color: white;
  }
  #box{
    background-color: #2f343c;
    color: white;
  }
  #nameInput{
    background-color: #21252b;
    color: white;
  }
  #kettei{
    background-color: #2f343c;
  }
  #kettei:hover{
    background-color: rgb(56, 165, 255);
    color: #2f343c;
  }
  /* スクロールの幅の設定 */
  ::-webkit-scrollbar {
    width: 10px;
    height: 10px;
  }

  /* スクロールの背景の設定 */
  ::-webkit-scrollbar-track {
    background-color: rgba(0,0,0,0);
  }

  /* スクロールのつまみ部分の設定 */
  ::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: #2f343c;
  }
  .messageBtn {
    color: #2f343c;
  }
  #messageBtns {
    background: #21252b;
  }
  .able {
    color: #818fae;
  }
}
</style>