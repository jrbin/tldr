<template>
  <div class="search">
    <input class="input" type="text" placeholder="Command Name" @input="updateQuery" :value="query" @blur="inputBlur" @keydown="onKeyUp">
    <div v-show="showDropdown && query.length > 0" class="search-dropdown">
      <div v-for="command in filteredCommands" :key="command" @mousedown.prevent="clickItem(command)" :class="['item', {'selected': selectedCommand===command}]">
        {{ command }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['commands'],
  data() {
    return {
      query: '',
      query2: '',
      showDropdown: false,
      selectedId: -1,
      lastQuery: '',
    }
  },
  computed: {
    filteredCommands() {
      return this.commands.filter(c => c.includes(this.query2)).sort((a, b) => a.length - b.length);
    },
    selectedCommand() {
      if (0 <= this.selectedId && this.selectedId < this.filteredCommands.length) {
        return this.filteredCommands[this.selectedId];
      }
      return '';
    }
  },
  methods: {
    updateQuery(e) {
      this.query = this.query2 = e.target.value;
      this.selectedId = this.filteredCommands.indexOf(this.query);
      this.showDropdown = true;
      if (this.query.length == 0) {
        this.lastQuery = '';
        this.$emit('inputEmpty');
      }
    },
    clickItem(command) {
      this.query = command;
      this.showDropdown = false;
      this.$emit('fetchDocs', command);
    },
    inputBlur() {
      this.showDropdown = false;
    },
    onKeyUp(e) {
      if ([38, 40, 9].includes(e.keyCode)) {
        // 40: arrow down
        // 38: arrow up
        // 9: tab
        e.preventDefault();
        if (this.filteredCommands.length === 0) {
          return;
        }
        if (e.keyCode === 40 || (e.keyCode == 9 && !event.shiftKey)) {
          this.selectedId = (this.selectedId+1) % this.filteredCommands.length;
        } else if (e.keyCode === 38 || (e.keyCode && event.shiftKey)) {
          if (this.selectedId <= 0) {
            this.selectedId = this.filteredCommands.length;
          }
          this.selectedId--;
        }
        this.query = this.selectedCommand;
      } else if (e.keyCode === 13) {
        // enter
        this.showDropdown = false;
        if (this.query.length > 0 && this.query !== this.lastQuery) {
          this.lastQuery = this.query;
          this.$emit('fetchDocs', this.query);
        }
      }
    }
  }
}
</script>

<style>
.search {
  position: relative;
  width: 100%;
  max-width: 100%;
}
.search-dropdown {
  position: absolute;
  top: 100%;
  overflow-x: hidden;
  overflow-y: auto;
  width: 100%;
  max-width: 100%;
  background: #fff;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  max-height: calc(100vh - 70px);
}
.search-dropdown .item {
  padding: 5px 10px;
  border-top: 1px solid #f0f0f0;
}
.search-dropdown .item.active,
.search-dropdown .item:hover {
  background-color: rgba(0, 0, 0, 0.05);
  color: rgba(0,0,0,0.95);
}
.search-dropdown .item.selected {
  font-weight: bold;
  background-color: rgba(0, 0, 0, 0.03);
  color: rgba(0,0,0,0.95);
}
</style>
