<template>
  <div class="home">
    <div class="container mx-auto my-12 text-left">
      <div class="formContainer my-12">
        <form action="" method="POST">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
            <div>
              <div class="form-group">
                <label for="topic_title" class="block mb-2">Topic *</label>
                <input
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="topic_title"
                  placeholder="Write your suggested topic here"
                  v-model="topic_title"
                />
              </div>
            </div>
            <div>
              <div class="form-group">
                <label for="target_level" class="block mb-2"
                  >Target level</label
                >
                <select
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="target_level"
                  v-model="target_level"
                >
                  <option value="beginner">Beginner</option>
                  <option value="medium">Medium</option>
                  <option value="advanced">Advanced</option>
                </select>
              </div>
            </div>
          </div>
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <div class="form-group">
                <label for="topic_details" class="block mb-2"
                  >More details *</label
                >
                <textarea
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="topic_details"
                  placeholder="Write your topic in more details here"
                  v-model="topic_details"
                ></textarea>
              </div>
            </div>
            <div>
              <div class="form-group">
                <label for="expected_result" class="block mb-2"
                  >Expected results</label
                >
                <textarea
                  class="form-control w-full p-2 border border-gray-300 rounded"
                  name="expected_result"
                  placeholder="Write what do you expect after watching this video"
                  v-model="expected_result"
                ></textarea>
              </div>
            </div>
          </div>

          <button
            type="submit"
            class="btn btn-success mt-6 bg-green-500 text-white py-2 px-4 rounded"
            @click.prevent="submitRequest()"
          >
            Send video request
          </button>
        </form>
      </div>
      <hr class="my-12" />
      <h1 class="mb-8 text-3xl font-bold">List of requested videos</h1>
      <div class="flex justify-between mb-4">
        <div class="inline-flex space-x-2" role="group">
          <button
            type="button"
            class="bg-blue-500 text-white py-2 px-4 rounded-l hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
            @click.prevent="getVideos('newFirst')"
            :class="{
              'bg-blue-800': sortBy === 'newFirst',
              'bg-blue-500': sortBy !== 'newFirst',
            }"
          >
            New first
          </button>
          <button
            type="button"
            class="bg-blue-500 text-white py-2 px-4 rounded-r hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
            @click.prevent="getVideos('topVotedFirst')"
            :class="{
              'bg-blue-800': sortBy === 'topVotedFirst',
              'bg-blue-500': sortBy !== 'topVotedFirst',
            }"
          >
            Top voted first
          </button>
        </div>
        <div>
          <input
            type="search"
            class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            id="search_box"
            placeholder="Type your search term"
            v-model="searchTerm"
          />
        </div>
      </div>

      <div id="listOfRequests">
        <template v-if="videos.length">
          <div
            class="card mb-6 border border-gray-300 rounded shadow"
            v-for="video in videos"
            :key="video._id"
          >
            <div class="card-body flex justify-between p-4">
              <div class="flex flex-col">
                <h3 class="text-xl font-semibold">{{ video.topic_title }}</h3>
                <p class="text-gray-600 mb-2">{{ video.topic_details }}</p>
                <p class="text-gray-600">
                  <strong>Expected results:</strong> {{ video.expected_result }}
                </p>
              </div>
              <div class="flex flex-col items-center">
                <a
                  class="btn btn-link cursor-pointer"
                  @click.prevent="vote(video._id, 'ups')"
                  :class="{
                    'opacity-50 pointer-events-none': video.votes.ups.includes(
                      $route.query.id
                    ),
                  }"
                  >🔺</a
                >
                <h3 class="text-xl font-semibold">
                  {{ video.votes.ups.length - video.votes.downs.length }}
                </h3>
                <a
                  class="btn btn-link cursor-pointer"
                  @click.prevent="vote(video._id, 'downs')"
                  :class="{
                    'opacity-50 pointer-events-none':
                      video.votes.downs.includes($route.query.id),
                  }"
                  >🔻</a
                >
              </div>
            </div>
            <div class="card-footer flex justify-between p-4 bg-gray-100">
              <div>
                <span class="text-blue-500">{{ video.status }}</span>
                &bullet; added by <strong>{{ video.author_name }}</strong> on
                <strong>{{ video.submit_date }}</strong>
              </div>
              <div class="ml-auto">
                <div
                  class="badge bg-green-500 text-white py-1 px-2 rounded capitalize"
                >
                  {{ video.target_level }}
                </div>
              </div>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="alert alert-info">No video requests found</div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from "@/components/HelloWorld.vue";
import axios from "axios";
export default {
  name: "HomeView",
  data: () => ({
    videos: [],
    author_name: "",
    author_email: "",
    topic_title: "",
    target_level: "beginner",
    topic_details: "",
    expected_result: "",
    sortBy: "newFirst",
    searchTerm: "",
  }),
  async mounted() {
    await this.getVideos();
  },
  methods: {
    resetForm() {
      this.topic_title = "";
      this.target_level = "beginner";
      this.topic_details = "";
      this.expected_result = "";
    },
    async submitRequest() {
      try {
        await axios.post("http://localhost:7777/video-request", {
          topic_title: this.topic_title,
          target_level: this.target_level,
          topic_details: this.topic_details,
          expected_result: this.expected_result,
          author_id: this.$route.query.id,
        });
        await this.getVideos();
        this.resetForm();
      } catch (error) {
        console.error(error);
      }
    },
    async getVideos(sortBy = "newFirst") {
      try {
        const response = await axios.get(
          `http://localhost:7777/video-request?sortBy=${sortBy}&searchTerm=${this.searchTerm}`
        );
        this.videos = response.data;
        this.sortBy = sortBy;
      } catch (error) {
        console.error(error);
      }
    },

    async vote(videoId, type) {
      try {
        await axios.put(`http://localhost:7777/video-request/vote`, {
          id: videoId,
          vote_type: type,
          user_id: this.$route.query.id,
        });
        await this.getVideos(this.sortBy);
      } catch (error) {
        console.error(error);
      }
    },
  },
  watch: {
    searchTerm: function () {
      this.getVideos();
    },
  },
};
</script>
