<template>
  <div>
    <div class="row full-height full-width">
      <div class="content">
        <div class="row">
          <div class="col-12">
            <div class="row page-header">
              <img src="./assets/prosa.jpg" width="50px" height="50px">
              <h4 class="title">Kanban Prosa</h4>
            </div>
          </div>
          <div class="col-12">
            <kanban
              :stages="stages"
              :cards="cards"/>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import kanban from './components/kanban.vue'
import refactor from './card-issues/refactor'
import createApi from './card-issues/createApi'
import improveAccuracy from './card-issues/improveAccuracy'
import twoFactorDesign from './card-issues/twoFactorDesign'

export default {
  name: 'App',
  components: {
    kanban
  },
  data() {
    return {
      stages: ['backlog', 'to_do', 'done'],
      cards: [
        refactor,
        createApi,
        improveAccuracy,
        twoFactorDesign
      ],
    };
  }
}
</script>

<style>
  .content {
    padding-left: 20px;
    width: 100vw;
  }
  .full-height {
    height: 100vh;
  }
  .full-width {
    width: 100vw;
  }
  .page-header {
    padding: 15px 0 10px 10px;
  }
  .title {
    padding: 10px 0 0 10px;
    font-weight: 800;
  }
</style>
