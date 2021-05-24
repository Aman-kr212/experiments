<template>
  <div id="app">
    <div v-if="!wordDetails.length" class="loading">Please Wait...</div>
    <table v-else id="table">
      <thead>
        <tr>
          <th>Word</th>
          <th>Count</th>
          <th>Pos</th>
          <th>Synonym</th>
        </tr>
      </thead>
      <tr v-for="word in wordDetails" :key="word.text">
        <td>{{ word.text }}</td>
        <td>{{ word.count }}</td>
        <td>{{ word.pos }}</td>
        <td>{{ word.syn }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
import Axios from "axios";
import { doc } from "./doc"; // file stored in local, since server's sending CORS error

export default {
  name: "App",

  data() {
    return {
      topTenWords: [],
      wordDetails: [],
    };
  },

  created() {
    this.getWordCounts();
  },

  methods: {
    /**
     * Separate the words from the document
     * count number of occurrences for each word
     * sort in desc order & select top 10
     */
    getWordCounts() {
      let wordCount = {},
        wordArray = [];
      const words = doc.match(/\b(\w+)\b/g);
      words.forEach((word) => {
        wordCount[word] = ++wordCount[word] || 1;
      });

      for (let key in wordCount) {
        wordArray.push({ text: key, count: wordCount[key] });
      }
      wordArray.sort((a, b) => b.count - a.count);
      this.topTenWords = [...wordArray.slice(0, 10)];
      this.fetchWordDetails();
    },

    /**
     * For the top 10 words, fetch their Parts of Speech(Pos) & synonyms
     */
    async fetchWordDetails() {
      const key =
        "dict.1.1.20210216T114936Z.e4989dccd61b9626.373cddfbfb8a3b2ff30a03392b4e0b076f14cff9";
      const url = `https://dictionary.yandex.net/api/v1/dicservice.json/lookup?key=${key}&lang=en-en`;

      const requests = this.topTenWords.map((word) => {
        return Axios.get(url + `&text=${word["text"]}`)
          .then((res) => {
            const response = res.data.def;
            if (response.length) {
              this.wordDetails.push({
                text: word["text"],
                pos: response[0].pos,
                syn:
                  response[0].tr.length && response[0].tr[0].syn
                    ? response[0].tr[0].syn[0].text
                    : "NA",
                count: word["count"],
              });
            } else {
              this.wordDetails.push({
                text: word["text"],
                pos: "NA",
                syn: "NA",
                count: word["count"],
              });
            }
          })
          .catch((err) => this.handleError(err));
      });
      await Promise.all(requests);
      // sort the received words based on occurrences
      this.wordDetails.sort((a, b) => b.count - a.count);
    },

    handleError(error) {
      console.log(error);
    },
  },
};
</script>

<style>
.loading {
  display: flex;
  height: calc(100vh - 16px);
  width: 100%;
  font-size: 2rem;
  color: darkgray;
  align-items: center;
  justify-content: center;
}

#table {
  width: 100%;
  background-color: #eee;
  border-collapse: collapse;
}

td,
th {
  line-height: 2;
  text-transform: capitalize;
}

tr {
  border-bottom: 1px solid #ddd;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
