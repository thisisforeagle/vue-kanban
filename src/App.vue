<template>
  <div id="app">
    <section class="section">
      <h4>Vue.js Kanban Board</h4>
      <a class="btn btn-success" data-toggle="modal" data-target="#newModal">+ Add new block</a>
    </section>
    <Kanban :stages="statuses" :blocks="blocks" @update-block="updateBlock">
      <div v-for="stage in statuses" :slot="stage" :key="stage">
        <h2>
          {{ stage }}
          <a>+</a>
        </h2>
      </div>
      <div v-for="item in blocks" :slot="item.id" :key="item.id">
        <div class="title">
          <strong>{{ item.title }}</strong>
        </div>
        <div class="description">
          {{ item.description }}
        </div>
      </div>
    </Kanban>
    <div class="modal fade" id="newModal" tabindex="-1" role="dialog" aria-labelledby="newModalLabel" aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content m-3">
          <div class="modal-body">
            <h3>Add a New Task</h3>
            <form>
              <div class="form-group">
                <label for="exampleInputEmail1">Title</label>
                <input v-model="newblock.title" type="text" class="form-control" id="exampleInputEmail1" placeholder="Give the task a name">
              </div>
              <div class="form-group">
                <label for="Description">Description</label>
                <textarea v-model="newblock.description" type="text" class="form-control" id="Description" placeholder="Describe what needs to be done"></textarea>
              </div>
            </form>
            <button class="btn btn-default" @click="addBlock">Save</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import faker from 'faker';
import { debounce } from 'lodash';
import Kanban from './components/Kanban';

export default {
  name: 'app',
  components: {
    Kanban,
  },
  data() {
    return {
      statuses: ['new', 'in-progress', 'needs-review', 'completed'],
      blocks: [],
      newblock: {
        title: '',
        description: '',
      },
    };
  },
  mounted() {
    if (window.localStorage.KanbanBlocks) {
      this.blocks = JSON.parse(window.localStorage.KanbanBlocks);
    }
  },

  methods: {
    updateBlock: debounce(function (id, status) {
      this.blocks.find(b => b.id === Number(id)).status = status;
      this.store();
    }, 500),
    addBlock: debounce(function () {
      this.blocks.push({
        id: this.blocks.length,
        status: 'new',
        title: this.newblock.title,
        description: this.newblock.description,
      });
      this.clearNewBlock();
      this.store();
    }, 500),
    store() {
      window.localStorage.KanbanBlocks = JSON.stringify(this.blocks);
    },
    clearNewBlock() {
      this.newblock = {
        title: '',
        description: '',
      };
    },
  },
};
</script>

<style lang="scss">
  @import './assets/kanban.scss';
  @import './assets/main.scss';
</style>
