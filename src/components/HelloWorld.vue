<template>
  <div class="login-container">
    <h1>Login Page</h1>
    <button  @click="loginWithGoogle">Login with Google</button>
    <button style="margin-top: 30px" @click="fetchData">Fetch Data</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "HelloWorld",
  data() {
    return {
      authenUrl: null,
      isLogged: false
    }
  },
  methods: {
    async loginWithGoogle() {
      if (!this.authenUrl) {
        await this.initAuthenUrl();
      }
      window.location.href = this.authenUrl; // Redirect to Google OAuth
    },
    async initAuthenUrl() {
      try {
        localStorage.removeItem("access_token");
        const {data} = await axios.get('http://localhost:8081/auth/url'); 
        this.authenUrl = data.url;
      } catch (error) {
        console.log(error);
      }
    },
    async initAccessToken(code) {
      try {
        const {data} = await axios.get(`http://localhost:8081/auth/callback?code=${code}`); 
        localStorage.setItem("access_token", (data.token))
      } catch (error) {
        console.log(error);
      }
    },
    async fetchData() {
      try {
        const {data} = await axios.get(`http://localhost:8081/private/messages`, {
          headers: {
            "Authorization": "Bearer " + localStorage.getItem("access_token")
          }
        }); 
        alert(data.message)
      } catch (error) {
        alert("UNAUTHORIZED");
        localStorage.removeItem("access_token");
      }
    }
  },
  created() {
    let accessToken = localStorage.getItem("access_token");
    if (accessToken) {
      return;
    }
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get('code');
    if (code) {
      this.initAccessToken(code);
    } else {
      this.initAuthenUrl();
    }
  }
};
</script>

<style>
.login-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

button {
  padding: 10px 20px;
  background-color: #4285f4;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #357ae8;
}
</style>
