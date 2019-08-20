<template>
  <div class="drag-container">
    <ul class="drag-list">
      <li v-for="stage in stages" class="drag-column" :class="{['drag-column-' + stage]: true}" :key="stage">
        <span class="drag-column-header">
          <h2>{{ stage }} ({{ count(stage) }})</h2>
          <div class="inline-loader" :class="{ 'loader': isLoading }"></div>
        </span>
        <div class="drag-options"></div>
        <ul class="drag-inner-list" ref="list" :data-status="stage">
          <li class="drag-item" v-for="block in getBlocks(stage)" :data-block-id="block.id" :key="block.id">
            <slot :name="block.id">
              <strong>{{ block.status }}</strong>
              <div>{{ block.id }}</div>
            </slot>
          </li>
        </ul>
        <div class="drag-column-footer">
          <slot :name="`footer-${stage}`"></slot>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
  import dragula from 'dragula';

  export default {
    name: 'KanbanBoard',

    props: {
      stages: Array,
      blocks: Array,
      isLoading: Boolean,
      stage: String
    },
    data() {
      return {
        newblocks: [],
        inprogressblocks: [],
        needsreviewblocks: [],
        completedblocks: [],
      };
    },
    watch: {
      blocks () {
        console.log('Blocks from parent: ',this.blocks);
      },
      isLoading () {
        console.log('Loading: ', this.isLoading);
      }
    },
    computed: {
      localBlocks() {
        return this.blocks;
      },
    },
    methods: {
      getBlocks(status) {
        let result = [];
        result = this.blocks.filter(block => block.status === status);
        
        return result;
      },
      editBlock (block) {
        this.$emit('editBlock', block);
      },
      groupBlocks () {
        this.newblocks = _.groupBy(this.blocks, 'new');
        this.inprogressblocks = _.groupBy(this.blocks, 'in-progress');
        this.needsreviewblocks = _.groupBy(this.blocks, 'needs-review');
        this.completedblocks = _.groupBy(this.blocks, 'completed');
      },
      count(status) {
        if (this.blocks.length > 0) {
          const count = _.groupBy(this.blocks, (block) => { block.status == status });
          console.log(count);
          console.log('Status: '+ status +'\nCount: ',count[Object.keys(count)[0]]);
          return count[Object.keys(count)[0]].length;
        }
      }
    },

    mounted() {
      dragula(this.$refs.list)
        .on('drag', (el) => {
          el.classList.add('is-moving');
        })
        .on('drop', (block, list) => {
          let index = 0;
          for (index = 0; index < list.children.length; index += 1) {
            if (list.children[index].classList.contains('is-moving')) break;
          }
          this.$emit('update-block', block.dataset.blockId, list.dataset.status, index);
        })
        .on('dragend', (el) => {
          el.classList.remove('is-moving');

          window.setTimeout(() => {
            el.classList.add('is-moved');
            window.setTimeout(() => {
              el.classList.remove('is-moved');
            }, 600);
          }, 100);
        });
    },
  };
</script>
