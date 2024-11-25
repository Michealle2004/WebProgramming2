<template>
  <div>
    <h1>Words Note for Language Learners</h1>
    
    <!-- Search bar -->
    <div class="ui fluid input" style="margin-bottom: 1em;">
      <input 
        type="text" 
        placeholder="Search words..." 
        v-model="searchQuery" 
      />
    </div>

    <!-- Language Filter Buttons -->
    <div style="margin-bottom: 1em;">
      <label>Show languages: </label>
      <button 
        class="ui button" 
        :class="{ active: selectedLanguages.includes('english') }"
        @click="toggleLanguage('english')"
      >
        English
      </button>
      <button 
        class="ui button" 
        :class="{ active: selectedLanguages.includes('german') }"
        @click="toggleLanguage('german')"
      >
        German
      </button>
      <button 
        class="ui button" 
        :class="{ active: selectedLanguages.includes('vietnamese') }"
        @click="toggleLanguage('vietnamese')"
      >
        Vietnamese
      </button>
      <button 
        class="ui button" 
        :class="{ active: selectedLanguages.length === 0 }"
        @click="selectedLanguages = []"
      >
        All
      </button>
    </div>

    <!-- Words Table -->
    <table id="words" class="ui celled compact table">
      <thead>
        <tr>
          <th v-if="selectedLanguages.length === 0 || selectedLanguages.includes('english')">English</th>
          <th v-if="selectedLanguages.length === 0 || selectedLanguages.includes('german')">German</th>
          <th v-if="selectedLanguages.length === 0 || selectedLanguages.includes('vietnamese')">Vietnamese</th>
          <th colspan="3"></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(word, i) in filteredWords" :key="i">
          <td v-if="selectedLanguages.length === 0 || selectedLanguages.includes('english')">{{ word.english }}</td>
          <td v-if="selectedLanguages.length === 0 || selectedLanguages.includes('german')">{{ word.german }}</td>
          <td v-if="selectedLanguages.length === 0 || selectedLanguages.includes('vietnamese')">{{ word.vietnamese }}</td>
          <td width="75" class="center aligned">
            <router-link :to="{ name: 'show', params: { id: word._id }}">Show</router-link>
          </td>
          <td width="75" class="center aligned">
            <router-link :to="{ name: 'edit', params: { id: word._id }}">Edit</router-link>
          </td>
          <td 
            width="75" 
            class="center aligned" 
            @click.prevent="onDestroy(word._id)"
          >
            <a :href="'/words/${word._id}'">Destroy</a>
          </td>
        </tr>
        <!-- Show 'Not Found' when no words match -->
        <tr v-if="filteredWords.length === 0">
          <td colspan="6" class="center aligned">Not Found</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';

export default {
  name: 'words',
  data() {
    return {
      words: [], // List of all words
      searchQuery: '', // Search input value
      selectedLanguages: [], // Selected languages to display (empty = all)
    };
  },
  computed: {
    filteredWords() {
      const query = this.searchQuery.toLowerCase();

      // Filter words based on search and selected languages
      return this.words.filter(word => {
        const matchesSearch =
          word.english.toLowerCase().includes(query) ||
          word.german.toLowerCase().includes(query) ||
          word.vietnamese.toLowerCase().includes(query);

        const matchesLanguage =
          this.selectedLanguages.length === 0 ||
          this.selectedLanguages.some(lang => word[lang]);

        return matchesSearch && matchesLanguage;
      });
    }
  },
  methods: {
    toggleLanguage(language) {
      if (this.selectedLanguages.includes(language)) {
        this.selectedLanguages = this.selectedLanguages.filter(lang => lang !== language);
      } else {
        this.selectedLanguages.push(language);
      }
    },
    async onDestroy(id) {
      const sure = window.confirm('Are you sure?');
      if (!sure) return;
      await api.deleteWord(id);
      this.flash('Word deleted successfully!', 'success');
      this.words = this.words.filter(word => word._id !== id);
    },
  },
  async mounted() {
    // Fetch words from the API when the component mounts
    this.words = await api.getWords();
  }
};
</script>
