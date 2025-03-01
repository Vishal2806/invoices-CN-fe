<template>
    <div class="container">
      <div class="login-box">
        <div class="logo-container">
          <div class="logo-text">
            <span class="text-black">CODE</span><span class="text-green">NICELY</span>
          </div>
        </div>
        <p class="auth-tagline">Secure & Seamless Authentication</p>
  
        <label>Email</label>
        <input v-model="email" type="text" placeholder="Enter Your Email" class="input-field" />
  
        <label>Password</label>
        <input v-model="password" type="password" placeholder="Enter Your Password" class="input-field" />
  
        <button class="login-button" @click="login">Login</button>
  
        <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  
  export default {
    data() {
      return {
        email: "",
        password: "",
        errorMessage: "",
      };
    },
    methods: {
      async login() {
        try {
          const response = await axios.post("http://localhost:5000/api/login", {
            email: this.email,
            password: this.password,
          });
  
          localStorage.setItem("token", response.data.token); // Save token
          this.$router.push("/dashboard"); // Redirect to dashboard
        } catch (error) {
          this.errorMessage = "Invalid email or password.";
        }
      },
    },
  };
  </script>
  
  <style scoped>
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #ffffff;
  }
  
  .login-box {
    background: white;
    padding: 20px;
    width: 320px;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    text-align: center;
  }
  
  .logo-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;
  }
  
  .logo-text {
    font-size: 24px;
    font-weight: bold;
  }
  
  .text-black {
    color: black;
  }
  
  .text-green {
    color: #4caf50;
  }
  
  .input-field {
    width: 100%;
    padding: 10px;
    margin-bottom: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
    color: black;
  }
  
  .login-button {
    margin-top: 15px;
    width: 100%;
    padding: 10px;
    background-color: #4285f4;
    color: white;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  .login-button:hover {
    background-color: #357ae8;
  }
  
  .auth-tagline {
    font-size: 14px;
    color: #666;
    margin-top: -5px;
    margin-bottom: 15px;
    font-weight: 500;
  }
  
  .error-message {
    color: red;
    font-size: 14px;
    margin-top: 10px;
  }
  </style>
  