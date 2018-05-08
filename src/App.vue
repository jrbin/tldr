<template>
  <div id="app" class="container">
    <div class="level">
      <div class="level-item is-narrow shrink">
        <h1 class="is-size-5">TLDR</h1>
      </div>
      <div class="level-item">
        <search-input
          :commands="commands"
          @fetchDocs="fetchDocs"
          @inputEmpty="notFound=false">
        </search-input>
      </div>
    </div>
    <div v-if="loading" class="content more-padding">
      <h1>Loading...</h1>
    </div>
    <div v-else-if="docs && 'length' in docs && docs.length > 0">
      <div class="tabs">
        <ul>
          <li v-for="doc in docs" :key="doc.id" :class="{'is-active': doc.active}">
            <a @click="setActive(doc)">{{doc.category}}</a>
          </li>
        </ul>
      </div>
      <div class="content" v-html="activeContent"></div>
    </div>
    <div v-else-if="notFound" class="content more-padding">
      <h1>Oops! Command not found!</h1>
      <p>We looked and looked for it, but it's nowhere. Maybe you can help us find it?</p>
      <p>TLDR is a community effort, we need people like you to raise the bar and find missing commands, suggest editions, and propose new pages.</p>
      <h3>How can I help?</h3>
      <p>Take a look at the open
        <a href="https://github.com/tldr-pages/tldr/issues?q=is%3Aissue+is%3Aopen+label%3A%22new+command%22">command requests</a>
        to throw a jab at things people need, or maybe join in on any of the open
        <a href="https://github.com/tldr-pages/tldr/pulls">command proposals</a>.
      </p>
      <p>If the command you want hasn't been proposed yet, feel encouraged to submit a proposal yourself! 😉 —
        <a href="https://github.com/tldr-pages/tldr/blob/master/CONTRIBUTING.md">start here</a>
      </p>
    </div>
    <div v-else class="content more-padding">
      <h1>Welcome!</h1>
      <p>We tried to learn other languages, but since we didn't find a TLDR for them it might not say "Welcome". We extend our deepest
          apologies.
      </p>
      <h3>How do I use this thing?</h3>
      <p>See the input box by the logo? Just type in a command and see the magic happen!</p>
      <p>Try
          <code>say</code>,
          <code>du</code>, or simply
          <code>man</code>.</p>
      <p>Some commands are widely available with the same interface, some other have variants per operating system. Currently the
          <code>tldr-pages</code> project splits commands into 4 categories: common, linux, osx, and sunos.</p>
      <p>
          <code>du</code>, for example, is available under both
          <code>linux</code> and
          <code>osx</code>.</p>
      <h3>What is TLDR ?</h3>
      <p>This is a web client for a project called
          <code>tldr-pages</code>; they are a community effort to simplify the beloved man pages with practical examples.</p>
      <p>Nifty indeed.</p>
      <p>Read more and join the tldr wagon at
          <a href="https://tldr-pages.github.io/">tldr-pages.github.io</a>
      </p>
      <h3>Do you have any unwanted pieces of trivia for me?</h3>
      <p>Well, this small app was built with ES6, and all the rendering is done with Vue.js.
      </p>
      <p>Have a
          <a href="https://github.com/jrbin/tldr">look inside</a> and feel free to
          <a href="https://github.com/jrbin/tldr/fork">fork</a>
      </p>
    </div>
    <modal v-show="showModal" @close="showModal = false">
      <template slot="head">
        Failed to fetch data
      </template>
      <p>Emm... Something is wrong when fetching data from Github.</p>
      <p>It's probably because you have reached Github API's access rate limit.</p>
      <p>Please try again later.</p>
    </modal>
  </div>
</template>

<script>
import _ from "lodash";
import marked from "marked";
import Modal from "./Modal.vue";
import SearchInput from "./SearchInput.vue";

// test data
const categories = [
  {
    id: 0,
    name: "common",
    url: "",
    children: ["7z", "tar", "echo", "sed", "grep", "awk", "sort"],
  },
  {
    id: 1,
    name: "osx",
    children: ["du", "say"],
  },
  {
    id: 2,
    name: "linux",
    children: ["du", "adduser"]
  }
]

function fetchStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return Promise.resolve(response);
  } else {
    return Promise.reject(new Error(response.statusText));
  }
}

function fetchJson(response) {
  return response.json();
}

function fetchError(error) {
  console.log("Request failed", error);
}

export default {
  name: "app",
  components: { Modal, SearchInput },
  data() {
    return {
      docs: [],
      categories: categories,
      loading: false,
      showModal: false,
      notFound: false,
    };
  },
  computed: {
    activeContent() {
      let activeDoc = this.docs.find(doc => doc.active);
      if (activeDoc && "content" in activeDoc) {
        return activeDoc.content;
      }
      return "";
    },
    commands() {
      return this.categories.map(c => c.children).reduce((acc, cur) => _.union(acc, cur));
    }
  },
  created() {
    const url = "https://api.github.com/repos/tldr-pages/tldr/contents/pages";
    fetch(url)
      .then(fetchStatus)
      .then(fetchJson)
      .then(json => {
        this.categories = json.map((value, i) => {
          return {
            id: i,
            name: value.name,
            url: value.url,
            children: []
          };
        });
        for (let i = 0; i < this.categories.length; i++) {
          fetch(this.categories[i].url)
            .then(fetchStatus)
            .then(fetchJson)
            .then(json => {
              this.categories[i].children = json.map(value => {
                if (value.name.endsWith(".md")) {
                  return value.name.substring(0, value.name.length - 3);
                }
                return value.name;
              });
            })
            .catch(fetchError);
        }
      })
      .catch(error => this.showModal = true);
  },
  methods: {
    fetchDocs(command) {
      this.error = "";
      let requests = [];
      for (let i = 0, j = -1; i < this.categories.length; i++) {
        let category = this.categories[i];
        if (!category.children.includes(command)) {
          continue;
        }
        j++;
        let categoryName = encodeURIComponent(category.name);
        command = encodeURIComponent(command);
        let url = `https://api.github.com/repos/tldr-pages/tldr/contents/pages/${categoryName}/${command}.md`;
        let request = fetch(url)
          .then(fetchStatus)
          .then(fetchJson)
          .then(json => ({
            id: i,
            category: category.name,
            content: marked(atob(json.content), { sanitized: true }),
            active: j == 0 ? true : false
          }));
        requests.push(request);
      }
      if (requests.length === 0) {
        this.docs = [];
        this.notFound = true;
        return;
      }
      this.notFound = false;
      this.loading = true;
      Promise.all(requests)
        .then(values => {
          values.sort((a, b) => a.id - b.id);
          this.docs = values;
        })
        .catch(error => {
          this.showModal = true;
        })
        .finally(() => {
          this.loading = false;
        });
    },
    setActive(activeDoc) {
      this.docs.forEach(doc => (doc.active = doc === activeDoc));
    }
  }
};
</script>

<style>
#app {
  max-width: 640px;
  padding-top: 30px;
  padding-bottom: 40px;
}
.level-item.shrink {
  flex-grow: 0;
  padding-left: 10px;
  padding-right: 30px;
}
.content.more-padding {
  padding-top: 10px;
}
</style>
