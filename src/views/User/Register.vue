<template>
  <div id="main">
    <b-alert v-model="showDismissibleAlert" variant="danger" dismissible>
      {{ errMsg }}
    </b-alert>
    <div class="loaderContainer" v-if="isLoading">
      <hollow-dots-spinner
          :animation-duration="1000"
          :dot-size="15"
          :dots-num="3"
          color="#ff1d5e"
          class="loader"
      />
    </div>
    <div>
      <h4>Start Using Smail</h4>
      <p style="font-size: 15px">
        Join Smail, to write your own mail, send your mail and many more
      </p>
      <h5>Register With</h5>
      <div class="registerwith">
        <router-link to="/Dashboard/0" class="btn btn-link">
          <FacebookIcon></FacebookIcon>
          Facebook
        </router-link>
        <router-link to="/Dashboard/0" class="btn btn-link">
          <TwitterIcon></TwitterIcon>
          Twitter
        </router-link>
        <router-link to="/Dashboard/0" class="btn btn-link">
          <GoogleIcon></GoogleIcon>
          Google
        </router-link>
      </div>
      <div class="register-form">
        <textinp name="alt-email" type="email" v-model="Alt_Email" placeholder="Alternate Email"></textinp>
        <textinp name="username" type="text" v-model="Username" placeholder="Username"></textinp>
        <textinp name="password" type="password" v-model="Password" placeholder="Password"></textinp>
        <textinp name="confirm-password" type="password" v-model="ConfirmPassword"
                 placeholder="Confirm Password"></textinp>
        <div class="checkinput">
          <input type="checkbox" id="ConfirmPolicy" v-model="checkTerms">
          <small for="ConfirmPolicy">
            I have read and agree to the terms of service and privacy
            policy</small>
        </div>
        <button type="button" class="btn btn-outline-light" v-on:click="insertdata">
          <BIcon icon="plus" aria-hidden="true"></BIcon>
          Create Account
        </button>
      </div>
      <div class="alternate">
        <p class="norm">
          Already have a SpeedMail Account ?
          <router-link to="/user/login">
            Sign In
          </router-link>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import textinp from "@/components/TextInputGroup"
import {FacebookIcon, TwitterIcon, GoogleIcon} from 'vue-simple-icons'
import {HollowDotsSpinner} from 'epic-spinners'

export default {
  name: "Register",
  data() {
    return {
      Username: "",
      Password: "",
      Alt_Email: "",
      ConfirmPassword: "",
      isLoading: false,
      checkTerms: false,
      showDismissibleAlert: false,
      errMsg: "",
    }
  },
  components: {
    textinp,
    FacebookIcon,
    TwitterIcon,
    GoogleIcon,
    HollowDotsSpinner
  },
  methods: {
    insertdata() {
      if (this.Username === "" || this.Password === "" || this.Alt_Email === "" || this.ConfirmPassword === "") {
        this.errMsg = "Please fill all the form"
        this.showDismissibleAlert = true
        return;
      }
      if (this.Password != this.ConfirmPassword) {
        this.errMsg = "Password doesn't match"
        this.showDismissibleAlert = true
        return;
      }
      if (this.checkTerms != true) {
        this.errMsg = "Please check our terms of service"
        this.showDismissibleAlert = true
        return;
      }
      if (!RegExp(/^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,24}))$/).test(this.Alt_Email)) {
          this.errMsg= "Your Email is Invalid"
          this.showDismissibleAlert=true
          return
      }
      if (RegExp(/[^\p{L}\d\s@#]/u).test(this.Username)) {
        this.errMsg = "Username cannot contains any symbol"
        this.showDismissibleAlert = true
        return
      }
      this.isLoading = true;
      const msgBuffer = new TextEncoder().encode(this.Password);

      // hash the message
      crypto.subtle.digest('SHA-256', msgBuffer).then((hashBuffer)=>{
        // convert ArrayBuffer to Array
        const hashArray = Array.from(new Uint8Array(hashBuffer));
        // convert bytes to hex string
        const hashHex = hashArray.map(b => ('00' + b.toString(16)).slice(-2)).join('');
        fetch("https://speedwagonmailback.herokuapp.com/account/add", {
          method: "POST",
          body: JSON.stringify({
            usern: this.Username,
            pass: hashHex,
            altermail: this.Alt_Email
          }),
          headers: {
            "content-type": "application/json"
          }
        }).then(response => response.json()
        ).then(result => {
          if (result.status == "OK") {
            localStorage.user = JSON.stringify(result.content);
            this.$store.commit("changeuser")
            this.isLoading = false
            this.$router.push("/dashboard/")
          } else {
            this.errMsg = "Username already exist, please choose another username"
            this.showDismissibleAlert = true
          }
        }).finally(() => {
              this.isLoading = false;
            }
        )
      });
    }
  }
}
</script>

<style scoped lang=scss>
#main {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  height: 780px;
  background-color: $brown-400;
  color: white;
  margin: auto;
  padding: 10px;
  border: solid 1px rgba($color: gray, $alpha: 0.7);
  border-radius: 20px;
  text-align: center;
  position: relative;

  .loader {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .loaderContainer {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 100;
    background: rgba(black, 0.2);
  }

  input {
    background-color: rgba($color: white, $alpha: 0.8);
  }

  .btn-link {
    color: $gray-100;
  }

  h5 {
    font-size: 97%;
  }

  .spacer {
    margin: 1rem auto 2rem auto;
    background-color: rgba($color: gray, $alpha: 0.7);
    width: 90%;
    height: 1px;
  }

  .registerwith {
    display: flex;
    justify-content: space-evenly;
    margin-top: 20px;

  }

  .register-form {
    display: flex;
    padding: 20px;
    flex-direction: column;
    text-align: start;

    .form-check {
      display: flex;
      align-items: center;
    }

    .btn {
      margin-top: 10px;
      width: 200px;
      align-self: center;
    }
  }

  .alternate {
    display: flex;
    flex-direction: column;
    font-size: medium;
  }
}
</style>
