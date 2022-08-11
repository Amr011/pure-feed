<script>
import { ref } from 'vue'
import * as RSSParser from 'rss-parser'
import { feedUrls } from '../constants'
import moment from 'moment'

const parser = new RSSParser()

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },

  setup() {
    const data = ref(null)
    const loading = ref(true)
    const error = ref(null)

    return {
      data,
      loading,
      error,
    }
  },

  data() {
    return {
      user: 'shaide32',
      feedUrls,
      feedsData: [],
      isRTL: false,
      selectedFeedIndex: 0,
      selectedFeedArticleIndex: 0,
    }
  },

  computed: {
    selectedFeedArticles() {
      return this.feedsData[this.selectedFeedIndex]?.items || []
    },

    selectedFeedArticlesTitle() {
      return this.feedsData[this.selectedFeedIndex]?.title || ''
    },

    selectedArticle() {
      return this.selectedFeedArticles[this.selectedFeedArticleIndex] || {}
    },
  },

  watch: {
    // user(newUser, oldUser){
    //   console.log(newUser, oldUser);
    //   if(newUser !== oldUser) {
    //     this.debouncedFetchData(newUser);
    //   }
    // }
  },

  mounted() {
    this.fetchArticles()
  },

  methods: {
    async fetchArticles() {
      try {
        this.feedUrls.forEach(async (feedUrl) => {
          // a temporary workaround for cors issue
          const url = 'http://localhost:9090/' + feedUrl
          let feed = await parser.parseURL(url)
          console.log(feed)
          this.feedsData.push(feed)
        })
      } catch (e) {
        console.error('ERROR:', e)
      }
    },
    datediff(date) {
      const today = moment()
      const postDate = moment(date)
      return postDate.from(today)
    },
    reverseDir() {
      this.isRTL = !this.isRTL
    },
  },
}
</script>

<template>
  <div class="feed">
    <div class="feed-title-container">
      <ul>
        <li
          v-for="(feed, index) in feedsData"
          @click="selectedFeedIndex = index"
          :key="index"
          :data-index="index"
          :class="{ selected: selectedFeedIndex == index }"
        >
          {{ feed.title }}
          <button style="font-size:0.5rem" @click="reverseDir">RTL</button>
        </li>
      </ul>
    </div>

    <div class="feed-article-list-container">
      <ul>
        <li
          v-for="(article, index) in selectedFeedArticles"
          @click="selectedFeedArticleIndex = index"
          :data-index="index"
          :key="index"
          :class="{ selected: selectedFeedArticleIndex == index }"
        >
          <div class="feed-article-list-item-title" style="display:flex; justify-content:between; ">
            <div style="justify-content:left;">
              {{ datediff(article.pubDate) }}
            </div>
            <div style="justify-content:right; margin-left:20px;">
              {{ selectedFeedArticlesTitle }}
            </div>
          </div>
          <div class="feed-article-list-item-content">{{ article.title }}</div>
        </li>
      </ul>
    </div>

    <div class="feed-article-container">
      <div style="margin-bottom:2rem;">
        <h2>
          <a :href="selectedArticle.link" target="_blank">{{ selectedArticle.title }}</a>
        </h2>
        <span v-html="selectedArticle['content:encoded']"> </span>
        <span v-html="selectedArticle['content']"> </span>
      </div>
    </div>
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.feed {
  display: flex;
  flex-direction: row;
}

.feed-title-container {
  flex: 0 0 15%;
  border-right: 0px solid grey;
  padding: 0 5px;
  -webkit-box-shadow: 4px 0px 7px -1px rgb(0 0 0 / 44%);
  -moz-box-shadow: 4px 0px 7px -1px rgba(0, 0, 0, 0.44);
  box-shadow: 4px 0px 7px -1px rgb(0 0 0 / 44%);
  overflow-y: auto;
}

.feed-article-list-container {
  flex: 0 0 30%;
  border-right: 0px solid grey;
  -webkit-box-shadow: 4px 0px 7px -1px rgb(0 0 0 / 44%);
  -moz-box-shadow: 4px 0px 7px -1px rgba(0, 0, 0, 0.44);
  box-shadow: 4px 0px 7px -1px rgb(0 0 0 / 44%);
  overflow-y: auto;
}

.feed-article-container {
  flex: 1 1 auto;
  overflow-y: auto;
  text-align: left;
  padding: 20px 20px;
  box-sizing: border-box;
}

.feed-article-list-item-title {
  font-size: 12px;
  color: lightslategray;
  margin-bottom: 5px;
}

input {
  width: 30%;
  font-size: 20px;
  padding: 5px;
  border-radius: 5px;
}

table {
  margin: 20px auto;
  padding: 5px;
  width: 70%;
}

td,
th {
  text-align: left;
  padding: 5px;
}

h3 {
  margin: 40px 0 0;
}

ul {
  display: flex;
  flex-direction: column;
  list-style-type: none;
  padding: 0;
  text-align: left;
}

.feed-article-list-container li:hover,
.feed-title-container li:hover {
  background: lightgray;
}

.selected {
  background: lightgray;
}

.feed-title-container li {
  display: inline-block;
  padding: 8px 10px;
  cursor: pointer;
  border-radius: 5px;
  font-weight: 600;
  font-size: 14px;
}

.feed-article-list-container li {
  display: inline-block;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 18px;
  border-bottom: 1px solid lightgray;
  padding: 10px 15px;
}

a {
  color: #42b983;
}
</style>
