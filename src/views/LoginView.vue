<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" /> -->

    <div class="container mx-auto my-12 text-left">
      <div class="form-login w-full md:w-1/3 mx-auto pt-20">
        <form
          action="http://localhost:7777/users/login"
          method="POST"
          @submit.prevent="submitLogin()"
        >
          <div class="row">
            <div class="w-full">
              <div class="form-group mb-4">
                <label for="author_name" class="block mb-2">Your name *</label>
                <input
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="author_name"
                  placeholder="Write your name here"
                  type="text"
                  required
                  v-model="author_name"
                />
                <p
                  class="text-red-500 text-sm mt-1"
                  v-if="author_name && author_name.length < 3"
                >
                  this field is required
                </p>
              </div>
              <div class="form-group mb-4">
                <label for="author_email" class="block mb-2"
                  >Your email *</label
                >
                <input
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="author_email"
                  placeholder="Write your email here"
                  type="email"
                  required
                  v-model="author_email"
                />
                <p
                  class="text-red-500 text-sm mt-1"
                  v-if="author_email && author_email.length < 3"
                >
                  this field is required and should be a valid email
                </p>
              </div>
              <div class="form-group">
                <button class="w-full bg-blue-500 text-white py-2 px-4 rounded">
                  login
                </button>
              </div>
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from "@/components/HelloWorld.vue";
import axios from "axios";
export default {
  name: "LoginPage",
  data: () => ({
    author_name: "",
    author_email: "",
  }),
  methods: {
    async submitLogin() {
      try {
        const response = await axios.post("http://localhost:7777/users/login", {
          author_name: this.author_name,
          author_email: this.author_email,
        });
        const data = response.data;
        this.$router.push(`/?id=${data.id}`);
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>
