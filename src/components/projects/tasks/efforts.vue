<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { taskManager } from '@state/helpers'

export default {
  components: { ButtonLoading },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
    dropdown: {
      type: Boolean,
      required: false,
      default: false
    },
  },
  data() {
    return {
      efforts: [],
      currentPage: 0,
      totalPages: 1,
      searchTerm: '',
      searchLoading: false,
    }
  },
  watch: {
    
    statusTask(data){
      if ( data.item.name === 'effort.change' && this.task.uuid === data.item.uuid ){
        this.effort = data.item.object
      }
    },
  },
  
  methods: {
    ...taskManager,

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loading = true
        this.getTaskEfforts()
      }
    },

    getTaskEfforts() {
      Axios()
        .get(
          'project-templates/effort/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&search=' + 
            this.searchTerm +
            '&page=' +
            this.currentPage
        )
        .then((response) => {

          this.efforts = response.data.data
          this.searchLoading = false
          this.loading = false
        })
    },

    changeTaskEffort(effort) {
      
      this.actionTask({ name: 'effort.change', uuid: this.task.uuid, object: effort})

      Axios()
      .put(
        'tasks/' +
          this.task.uuid +
          '/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug,
        {
          config_issue_effort_id: effort.id,
        }).then((response) => {
          this.task.effort = effort
        })

    },
    search() {
      this.searchLoading = true
      this.types = []
      this.currentPage = 1
      this.getTaskEfforts()
    },
  }
}
</script>

<template>
<div>
  <span 
    v-if="task.effort !== null && !dropdown" 
    class="badge badge-light mr-2 mb-1">
    {{ task.effort.title }} - {{ task.effort.effort }} {{ $t('points') }}
  </span>
  <b-dropdown 
    v-if="dropdown && authorize('tasks', 'create')" 
    ref="dropdown" 
    left 
    class="styled-dropdown no-secondary" 
    @shown="loadMore">
    <template v-slot:button-content>
      <span class="badge badge-light mr-2">
        {{ task.effort.title }} - {{ task.effort.effort }} {{ $t('points') }}
        <font-awesome-icon 
        :icon="['far', 'angle-down']" 
        style="position:relative; margin-left:4px;top: 0px;" />
      </span>
    </template>
    <b-dropdown-form style="width:300px;">
      <b-input-group class="mb-1">
        <b-input-group-append class="wd-100">
          <b-form-input 
          v-model="searchTerm" 
          :placeholder="$t('Search type')"
          type="search" 
          autocomplete="off"
          size="sm"></b-form-input>
          <ButtonLoading
          :loading="searchLoading"
          type="btn-sm"
          mode="button"
          icon="search"
          @action="search"
          ></ButtonLoading>
        </b-input-group-append>
      </b-input-group>
    </b-dropdown-form>
    <div class="scroll-activate-h200 label-line">
      <b-link v-for="effort in efforts" 
        :key="effort.id" 
        class="dropdown-item project-label-line" 
        @click="changeTaskEffort(effort)">
        {{ effort.title }} - {{ effort.effort }} {{ $t('points') }}
      </b-link>
    </div>
  </b-dropdown>
</div>
</template>
