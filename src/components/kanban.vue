<template>
  <div>
    <b-modal
      v-model="show_modal"
      title="Add Task"
      size="sm"
      @show="resetForm"
      @hidden="resetForm">
      <div class="modal-form">
        <b-form-group>
          <label for="title">Title</label><br>
          <input
            v-model="form.title"
            type="text"
            name="title">
        </b-form-group>
        <b-form-group>
          <label for="assignee">Assignee</label><br>
          <input
            v-model="form.assignee"
            type="text"
            name="assignee">
        </b-form-group>
        <b-form-group>
          <label for="tags">Tags</label><br>
          <input
            v-model="form.tags"
            type="text"
            name="tags">
        </b-form-group>
        <b-form-group>
          <label for="start">Start Date</label><br>
          <input
            v-model="form.start_date"
            type="date"
            name="start">
        </b-form-group>
        <b-form-group>
          <label for="end">End Date</label><br>
          <input
            v-model="form.end_date"
            type="date"
            name="end">
        </b-form-group>
      </div>
      <div
        slot="modal-footer">
        <b-button
          class="button-modal-footer"
          variant="secondary"
          @click="hideModal">
          Cancel
        </b-button>
        <b-button
          :disabled="!(form.title && form.assignee && form.tags && form.start_date && form.end_date && form.end_date >= form.start_date)"
          class="add-task-btn"
          @click="addTask">
          Add Task
        </b-button>
      </div>
    </b-modal>
    <div class="stage-container">
      <div
        v-for="(stage, indexStage) in stages"
        :key="indexStage"
        class="stage">
        <div class="stage-header">
          <span class="stage-title">{{ convertTitle(stage) }}</span>
          <span
            class="add-task"
            @click="onClickAddTask(indexStage)">
            <b-icon-plus/>Add Task
          </span>
        </div>
        <draggable
          :list="local_board[indexStage]"
          :id="`issue-${indexStage}-${stage}`"
          class="list-group stage-card-container"
          group="tasks"
          @start="start"
          @end="end">
          <div
            v-for="(card, indexCard) in local_board[indexStage]"
            :key="indexCard"
            class="stage-card list-group-item">
            <div
              :id="`card-${card.issue_id}`"
              class="card-container">
              <div class="card-title-container">
                <span class="card-title">{{ card.title }}</span>
              </div>
            </div>
            <div class="bottom-line">
              <span class="card-assignee-container green">
                <span class="card-assignee">
                  {{ card.assignee.charAt(0).toUpperCase() }}
                </span>
              </span>
              <span
                :class="`card-tag ${getColor(card.tags)}`">
                {{ card.tags.toUpperCase() }}
              </span>
              <span class="card-remaining">
                {{ calculateDuration(card.start_date, card.end_date) }}
              </span>
            </div>
          </div>
        </draggable>
      </div>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import  { BIconPlus, BModal } from 'bootstrap-vue'

export default {
  name: 'kanban',
  props: {
    stages: {
      type: Array,
      default: () => [],
    },
    cards: {
      type: Array,
      default: () => [],
    },
  },
  components: {
    draggable,
    BIconPlus,
    BModal,
  },
  data() {
    return {
      local_board: [],
      selected_stage: null,
      show_modal: false,
      tags: ['Back End', 'Front End', 'Research', 'Design'],
      tags_color: ['red', 'black', 'orange', 'blue'],
      colors: ['red', 'orange', 'green', 'blue', 'purple', 'black'],
      form: {
        issue_id: 0,
        title: null,
        assignee: null,
        start_date: null,
        end_date: null,
        tags: null,
      },
    };
  },
  created() {
    this.stages.forEach((element) => {
      if (element === 'backlog') {
        this.local_board.push(this.cards);
      } else {
        this.local_board.push([]);
      }
    });
  },
  methods: {
    convertTitle(item) {
      const component = item.split('_');
      if (component.length > 1) {
        let title = '';
        component.forEach((element, index) => {
          if (index === component.length - 1) {
            title += element.charAt(0).toUpperCase() + element.slice(1);
          } else {
            title += `${element.charAt(0).toUpperCase() + element.slice(1)} `;
          }
        });
        return title;
      }
      return item.charAt(0).toUpperCase() + item.slice(1);
    },
    calculateDuration(start, end) {
      const startDate = new Date(start);
      const endDate = new Date(end);
      const duration = new Date(endDate - startDate);
      return `${duration.getDate()} days`;
    },
    getColor(tagName) {
      const idx = this.tags.findIndex(item => item === tagName);
      return this.tags_color[idx];
    },
    start(event) {
      const stages = document.getElementsByClassName('stage-card-container');
      // eslint-disable-next-line
      const indexFrom = Number(event.from.id.split('-')[1]);
      for (const index in stages) {
        const idx = Number(index);
        let stage;
        if (!isNaN(idx)) {
          stage = stages[Number(index)];
          if (idx !== indexFrom) {
            stage.style.minHeight = `${((stage.children.length + 1) * 150) - ((stage.children.length + 1) * 25)}px`;
            stage.style.minWidth = '180px';
          }
        }
      }
    },
    end(event) {
      // const message = `Do yo want to stop "${card.title} task record?"`;
      const stages = document.getElementsByClassName('stage-card-container');
      // eslint-disable-next-line
      const indexTo = Number(event.to.id.split('-')[1]);
      for (const index in stages) {
        const idx = Number(index);
        let stage;
        if (!isNaN(idx)) {
          stage = stages[Number(index)];
          if (idx !== indexTo) {
            stage.style.minHeight = '0px';
            stage.style.minWidth = '0px';
          }
        }
      }
      const idComponent = event.to.id.split('-');
      this.local_board[Number(idComponent[1])].forEach((element) => {
        // eslint-disable-next-line
        element.stage = idComponent[2];
      });
      this.$nextTick(() => {
        const stageTo = document.getElementById(event.to.id);
        stageTo.style.minHeight = '0px';
        stageTo.style.minWidth = '0px';
      });
    },
    onClickAddTask(idStage) {
      this.selected_stage = idStage;
      this.show_modal = true;
    },
    resetForm() {
      this.form = {
        issue_id: 0,
        title: null,
        assignee: null,
        start_date: null,
        end_date: null,
        tags: null,
      };
    },
    addTask() {
      const idx = this.tags.findIndex(item => item === this.form.tags);
      if (idx === -1) {
        this.tags.push(this.form.tags);
        this.tags_color.push(this.colors[Math.floor(Math.random() * 6)]);
      }
      this.local_board[this.selected_stage].push(this.form);
      this.selected_stage = null;
      this.show_modal = false;
      this.resetForm();
    },
    hideModal() {
      this.resetForm();
      this.show_modal = false;
    },
  },
}
</script>

<style scoped>
  .modal-form {
    text-align: center;
  }
  .button-modal-footer {
    margin-right: 10px;
  }
  .add-task-btn {
    background-color: #0cac90;
  }
  .stage-container {
    height: 100%;
  }
  .stage {
    display: inline-block;
    vertical-align: top;
    width: 250px;
    margin-right: 10px;
    box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.2);
    max-height: calc(100vh - 80px);
    padding: 15px;
    border-radius: 6px
  }
  .stage-title {
    font-size: 20px;
  }
  .add-task {
    background-color: #0cac90;
    font-size: 14px;
    padding: 5px 10px 5px 10px;
    color: white;
    border-radius: 15px;
    margin-right: 0;
    text-align: center;
    float: right;
    cursor: pointer;
    transition: 0.2s;
  }
  .add-task:hover {
    background-color: #088b75;
  }
  .stage-card-container {
    margin-top: 10px;
    max-height: calc(100vh - 140px);
    overflow-y: scroll;
    overflow-x: hidden;
    width: 220px;
  }
  ::-webkit-scrollbar {
    height: 12px;
    width: 8px;
  }
  ::-webkit-scrollbar-thumb {
    background-color: #e7e7e7;
    border-radius: 6px;
  }
  .stage-card {
    background: #fafafa;
    width: 205px;
    margin-bottom: 10px;
    box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.2);
    cursor: grab;
    border-radius: 5px;
  }
  .card-title {
    font-weight: 700;
  }
  .bottom-line {
    margin-top: 10px;
  }
  .card-assignee-container {
    display: inline-block;
    font-weight: 500;
    width: 25px;
    height: 25px;
    text-align: center;
    border-radius: 50%;
  }
  .red {
    color: rgba(255, 0, 0, 1);
    background-color: rgba(255, 0, 0, 0.15);
  }
  .orange {
    color: rgba(255, 200, 0, 1);
    background-color: rgba(255, 200, 0, 0.15);
  }
  .green {
    color: rgba(0, 255, 150, 1);
    background-color: rgba(0, 255, 0, 0.15);
  }
  .blue {
    color: rgba(0, 0, 255, 1);
    background-color: rgba(0, 0, 255, 0.15);
  }
  .purple {
    color: rgba(180, 0, 255, 1);
    background-color: rgba(180, 0, 255, 0.15);
  }
  .black {
    color: rgba(0, 0, 0, 1);
    background-color: rgba(0, 0, 0, 0.15);
  }
  .card-tag {
    font-size: 12.5px;
    margin-left: 10px;
    padding: 0 4px 1.5px 0;
    font-weight: 500;
  }
  .card-remaining {
    float: right;
  }
  .disabled {
    cursor: not-allowed;
  }
  .btn-secondary:disabled {
    background-color: #5ac7b5;
    border: 1px solid #5ac7b5;
  }
</style>