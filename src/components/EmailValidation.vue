<template>
  <div id="app">
  <form @submit.prevent="submitForm">
    <div class="email">
      <label for="email">Email Address:</label><br>
      <input 
        v-model.trim="email" 
        @blur="v$.email.$touch" 
        type="email" 
        placeholder="Enter your email address" 
        @keyup="onKeyup"
        required 
      />
    </div>
    <div class="container">
      <template v-if="v$.email.$dirty && v$.email.$errors.length">
        <span
          v-for="error of v$.email.$errors"
          :key="error.$uid"
        >
          <span class="error">{{ error.$message }}</span>
        </span>
      </template>
      <template v-else-if="emailMessage">
        <span class="error">{{emailMessage}}</span>
      </template>
    </div>
    <div class="submit">
      <button type="submit" :disabled="isDisabled"
        :class="[isDisabled ? '' : activeClass]" 
      >Submit</button>
    </div>
  </form>
</div>
</template>
<script>
import useVuelidate from "@vuelidate/core";
import { required, email } from "@vuelidate/validators";
import axios from 'axios';

export default {
  name: "EmailValidation",
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      email: "",
      activeClass: 'active',
      isDisabled: true,
      emailMessage: '',
      API_KEY: process.env.VUE_APP_API_KEY
    };
  },
  methods: {
    onSubmit() {
      this.email = '';
      this.isDisabled = true;
      alert('Thanks for registering with ryd pay! 🎉');
    },
    async getUser(email) {
      try {
        const baseURL = `https://emailvalidation.abstractapi.com/v1/?api_key=${this.API_KEY}&email=${email}`;
        const response = await axios.get(baseURL);

        if (response.data?.deliverability === "DELIVERABLE") {
          this.isDisabled = false;
          this.emailMessage = '';
        } else {
          this.emailMessage = `${email} is not a valid email address.`
        }
      } catch (error) {
        // TODO: Optimise serverside error handling
        console.error('error', error);
      }
    },
    onKeyup(){
      // only make API call after user has entered email and there are no validation errors
      const silentErrors = this.v$.email.$silentErrors.length;
      if (!silentErrors) {
        setTimeout(() => {
          this.getUser(this.email)
        }, 500)
      } else {
        this.isDisabled = true;
      }
      
    }
  },
  validations() {
    return {
      email: { required, email },
    };
  },
};
</script>

<style scoped>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  padding: 70px;
  max-width: 360px;
  font-size: 16px;
  margin: 0 auto;
  display: table;
  line-height: 2em;
  color: #2c3e50;
  margin-top: 25vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
 form {
  width: 300px;
  padding: 10px 40px;
  text-align: center;
}
 form label {
  text-transform: uppercase;
  font-size: 13px;
  letter-spacing: 0.03em;
  font-weight: bold;
}

 form input {
  border: 1px solid #ccc;
  color: #333;
  width: calc(100% - 30px);
}

 form input, form button {
  border-radius: 4px;
  padding: 8px 15px;
  font-family: 'Work Sans', sans-serif;
  font-weight: 300;
}

 button {
  color: rgb(133, 116, 116);
  background-color: whitesmoke;
  border: none;
  width: 100%;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  cursor: pointer;
}
 .active {
  background: #5968d7;
  color: white;
}

.error {
  font-size: small;
  color: red;
}

.container {
  height: 25px;
}

</style>
