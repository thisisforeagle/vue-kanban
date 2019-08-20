<template>
  <div id="app">
    <section class="section">
      <h1>Vue.js Kanban Board</h1>
      <a href="https://kanban-a82d.restdb.io/home/db/_user:5d584101ecdd522a00005b65/cards/5d58445aecdd522a00005f6a"
        target="_blank">Manage DB Items</a><br>
      <a href="https://kanban-a82d.restdb.io/rest/kanban" target="_blank">View items</a><br>
      <br>
      <a class="btn btn-success" data-toggle="modal" data-target="#modal">+ Add new block</a>
    </section>
    <Kanban :stages="statuses" :blocks="blocks" :is-loading="isLoading" @update-block="updateBlock" @edit-block="editBlock">
      <div v-for="stage in statuses" :stage="stage" :key="stage">
        <h2>
          {{ stage }}
          <a>+</a>
        </h2>
      </div>
      <div v-for="item in blocks" :slot="item.id" :key="item.id" @click="editBlock(item)">
        <div class="title">
          <div class="row">
            <div class="col-xs-8 title-container">
              <strong class="title-text">{{ item.title }}</strong><br>
              <i :title="item.date | date">{{ time_ago(item.date) }}</i>
            </div>
            <div class="col-xs-4">
              <span class="hours">{{ item.hours.split(' ')[0] }}</span>
            </div>
          </div> 
        </div>
        <div class="description">
            <span style="white-space: pre-line">{{ item.description }}</span>
        </div>
      </div>
    </Kanban>

    <div class="debug">
      <hr>
      <button class="btn btn-primary " type="button" data-toggle="collapse" data-target="#collapseExample"
        aria-expanded="false" aria-controls="collapseExample">
        Show JSON (DEBUG)
      </button>
      <div class="collapse" id="collapseExample">
        <div class="card card-body">
          <pre>{{ blocks }}</pre>
        </div>
      </div>
    </div>

    <div class="modal fade" id="modal" tabindex="-1" role="dialog" aria-labelledby="modalLabel"
      aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content m-3">
          <div class="modal-body">
            <h3 v-if="!isEditing">Add a Task</h3>
            <h3 v-if="isEditing">Edit/Delete Task</h3>
            <hr>
            <form>
              <div class="form-group">
                <label for="exampleInputEmail1">Title</label>
                <input v-model="newblock.title" type="text" class="form-control" id="exampleInputEmail1"
                  placeholder="Give the task a name">
              </div>
              <div class="form-group" v-if="isEditing">
                <label for="sel1">Change Status:</label>
                <select class="form-control" v-model="newblock.status">
                  <option v-for="status in statuses">{{ status }}</option>
                </select>
              </div>
              <div class="form-group">
                <label for="sel1">Estimated Time:</label>
                <select class="form-control" v-model="newblock.hours">
                  <option v-for="time in times">{{ time }} hours</option>
                </select>
              </div>
              <div class="form-group">
                <label for="Description">Description</label>
                <textarea v-model="newblock.description" type="text" class="form-control" id="Description"
                  placeholder="Describe what needs to be done"></textarea>
              </div>
            </form>
            <button class="btn btn-success" data-dismiss="modal" @click="addBlock" v-if="!isEditing">Save</button>
            <button class="btn btn-success" data-dismiss="modal" @click="update(newblock)" v-if="isEditing">Update</button>
            <button class="btn btn-danger" data-dismiss="modal" @click="deleteBlock(newblock)" v-if="isEditing">Delete</button>
            <button class="btn btn-default" data-dismiss="modal">Cancel</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { debounce } from 'lodash';
  import Kanban from './components/Kanban';
  import * as moment from 'moment';

  const API_KEY = '5d5845bef3894120866b5861';

  $.ajaxPrefilter((options) => {
    if (!options.beforeSend) {
      options.beforeSend = (xhr) => {
        xhr.setRequestHeader('x-apikey', API_KEY);
      }
    }
  });

  export default {
    name: 'app',
    components: {
      Kanban,
    },
    data() {
      return {
        statuses: ['new', 'in-progress', 'needs-review', 'completed'],
        times: [1,2,3,4,5,6,7,8],
        blocks: [],
        newblock: {
          date: new Date(),
          id: 0,
          title: '',
          description: '',
          status: 'new',
          hours: 0
        },
        isEditing: false,
        isLoading: true,
      };
    },
    filters: {
      date(value) {
        if (value) {
          return moment(String(value)).format('MM/DD/YYYY hh:mm')
        }
      }
    },
    watch: {
      isEditing() {
        console.log('isEditing: ',this.isEditing);
      }
    },
    methods: {
      updateBlock: debounce(function (id, status) {
        const blocktoupdate = this.blocks.find(b => b.id === Number(id));
        blocktoupdate.status = status;
        this.update(blocktoupdate);
      }, 500),
      addBlock: debounce(function () {
        this.newblock.id = this.blocks.length;
        this.newblock.date = new Date();

        this.blocks.push({
          id: this.blocks.length,
          status: this.newblock.status,
          title: this.newblock.title,
          description: this.newblock.description,
          hours: this.newblock.hours
        });

        this.store();
        $('#modal').modal('hide');
      }, 500),
      editBlock (block) {
        this.isEditing = true;
        $('#modal').modal('show');
        this.newblock = block;
      },
      async deleteBlock (block) {
        const res = await $.ajax({
          type: "DELETE",
          url: 'https://kanban-a82d.restdb.io/rest/kanban/' + block._id,
          contentType: "application/json"
        }).done((result) => {
          console.log("Deleted", result);
        });
        this.getBlocks();
      },
      async store() {
        const res = await $.ajax({
          type: "POST",
          url: 'https://kanban-a82d.restdb.io/rest/kanban',
          contentType: "application/json",
          data: JSON.stringify(this.newblock)
        }).done((result) => {
          console.log("Saved", result);
        });
        this.clearNewBlock();
        $('modal').hide();
        this.getBlocks();
      },
      update(block) {
        this.isLoading = true;
        $.ajax({
          type: "PUT",
          url: 'https://kanban-a82d.restdb.io/rest/kanban/' + block._id,
          contentType: "application/json",
          data: JSON.stringify(block)
        }).done((result) => {
          console.log("Updated", result);
          this.clearNewBlock();
          this.getBlocks();
        });
      },
      clearNewBlock() {
        this.newblock = {
          date: new Date(),
          id: 0,
          title: '',
          description: '',
          status: 'new',
          hours: 0,
        };
      },
      async getBlocks() {
        this.blocks = await $.getJSON('https://kanban-a82d.restdb.io/rest/kanban');
        this.isLoading = false;
      },
      time_ago(time) {

        switch (typeof time) {
          case 'number':
            break;
          case 'string':
            time = +new Date(time);
            break;
          case 'object':
            if (time.constructor === Date) time = time.getTime();
            break;
          default:
            time = +new Date();
        }
        var time_formats = [
          [60, 'seconds', 1], // 60
          [120, '1 minute ago', '1 minute from now'], // 60*2
          [3600, 'minutes', 60], // 60*60, 60
          [7200, '1 hour ago', '1 hour from now'], // 60*60*2
          [86400, 'hours', 3600], // 60*60*24, 60*60
          [172800, 'Yesterday', 'Tomorrow'], // 60*60*24*2
          [604800, 'days', 86400], // 60*60*24*7, 60*60*24
          [1209600, 'Last week', 'Next week'], // 60*60*24*7*4*2
          [2419200, 'weeks', 604800], // 60*60*24*7*4, 60*60*24*7
          [4838400, 'Last month', 'Next month'], // 60*60*24*7*4*2
          [29030400, 'months', 2419200], // 60*60*24*7*4*12, 60*60*24*7*4
          [58060800, 'Last year', 'Next year'], // 60*60*24*7*4*12*2
          [2903040000, 'years', 29030400], // 60*60*24*7*4*12*100, 60*60*24*7*4*12
          [5806080000, 'Last century', 'Next century'], // 60*60*24*7*4*12*100*2
          [58060800000, 'centuries', 2903040000] // 60*60*24*7*4*12*100*20, 60*60*24*7*4*12*100
        ];
        var seconds = (+new Date() - time) / 1000,
          token = 'ago',
          list_choice = 1;

        if (seconds == 0) {
          return 'Just now'
        }
        if (seconds < 0) {
          seconds = Math.abs(seconds);
          token = 'from now';
          list_choice = 2;
        }
        var i = 0,
          format;
        while (format = time_formats[i++])
          if (seconds < format[0]) {
            if (typeof format[2] == 'string')
              return format[list_choice];
            else
              return Math.floor(seconds / format[2]) + ' ' + format[1] + ' ' + token;
          }
        return time;
      },
      stylizePreElements () {
        var preElements = document.getElementsByTagName("pre");
        for (let i = 0; i < preElements.length; ++i) {
          var preElement = preElements[i];
          preElement.className += "prettyprint";
        }
      },
    },
    mounted() {
      this.getBlocks();
      //this.stylizePreElements();
      // add listener for modal close event
      $("#modal").on('hidden.bs.modal', () => {
        this.isEditing = false;
      });
      console.log('Underscore: ',_);
    },
  };
</script>

<style lang="scss">
  @import './assets/kanban.scss';
  @import './assets/main.scss';
</style>