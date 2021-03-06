<script>
import Axios from '@utils/axios'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'
import ButtonLoading from '@components/utils/button-loading'
import TitleLoading from '@components/utils/title-loading'

export default {
  components: {
    TitleLoading,
    ButtonLoading,
    Swatches,
  },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
  },
  data() {
    return {
      statusAssignLoading: false,
      loading: false,
      btnLoading: false,
      boxSelected: 'list',
      labels: [],
      label: this.getLabelDefault(),
      labelSelected: '',
      labelProps: { blank: true, width: 15, height: 15, class: 'm1' },
      alertMessage: '',
      alertStatus: false,
    }
  },
  watch: {
    labelSelected: function(val, oldval) {
      if (val === null) {
        let color = this.label.color
        this.label = this.getLabelDefault()
        this.label.color = color
      } else if (val.id) {
        this.label = val
      } else {
        let color = this.label.color
        this.label = this.getLabelDefault()
        this.label.title = val.title ? val.title : val
        this.label.color = color
      }
    },
  },
  methods: {
    getLabelDefault() {
      return {
        id: null,
        title: '',
        color: '#000000',
      }
    },

    getAllLabels() {
      this.loading = true
      this.alertStatus = false
      Axios()
        .get(
          'task-labels/not-added' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
        )
        .then((response) => {
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.labels = response.data.data.filter((item) => !this.task.labels.some((label) => label.id === item.id))
          
            if (!this.labels.length){
              this.openCreate()
            }
          }
          this.loading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },

    addLabel() {
      this.btnLoading = true
      Axios()
        .post(
          'task-labels/' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid,
          {
            color: this.label.color,
            title: this.label.title,
          }
        )
        .then((response) => {
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.task.labels.push(response.data.data)
            this.getAllLabels()
            this.label = this.getLabelDefault()
            this.labelSelected = ''
            this.boxSelected = 'list'
          }
          this.btnLoading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },

    editLabel(label) {
      this.btnLoading = true

      Axios()
        .put(
          'task-labels/' +
            label.id +
            '/' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid,
          {
            color: label.color,
            title: label.title,
          }
        )
        .then((response) => {
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.getAllLabels()
            this.labels = this.getLabelDefault()
            this.boxSelected = 'list'
          }
          this.btnLoading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },

    assignLabel(label) {
      if (!this.statusAssignLoading) {
        this.statusAssignLoading = true
        Axios()
          .post(
            'task-labels/' +
              '?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug +
              '&task_uuid=' +
              this.task.uuid,
            {
              color: label.color,
              title: label.title,
            }
          )
          .then((response) => {
            if (response.data.message) {
              this.alertMessage = 'Error. ' + response.data.message
              this.alertStatus = true
            } else {
              this.task.labels.push(response.data.data)
              this.getAllLabels()
              this.labels = this.getLabelDefault()
            }
            this.statusAssignLoading = false
          })
          .catch((error) => {
            this.alertMessage = 'Error. ' + error.response.data.message
            this.alertStatus = true
          })
      }
    },

    removeLabel(label) {

      Axios()
      .delete(
        'task-labels/' +
          label.id +
          '/detach/' +
          '?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&task_uuid=' +
          this.task.uuid
      )
      .then((response) => {
        if (response.data.message) {
          this.alertMessage = 'Error. ' + response.data.message
          this.alertStatus = true
        } else {
          this.getAllLabels()
          this.boxSelected = 'list'
          this.labels = this.getLabelDefault()
          this.task.labels.splice(this.task.labels.indexOf(label), 1)
        }
      })
      .catch((error) => {
        this.alertMessage = 'Error. ' + error.response.data.message
        this.alertStatus = true
      })

    },
    deleteLabel(label) {
      this.boxSelected = ''

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
              
          Axios()
          .delete(
            'task-labels/' +
              label.id +
              '/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug +
              '&task_uuid=' +
              this.task.uuid
          )
          .then((response) => {
            if (response.data.message) {
              this.alertMessage = 'Error. ' + response.data.message
              this.alertStatus = true
            } else {
              this.getAllLabels()
              this.labels = this.getLabelDefault()
              this.task.labels.splice(this.task.labels.indexOf(label), 1)
            }
          })
          .catch((error) => {
            this.alertMessage = 'Error. ' + error.response.data.message
            this.alertStatus = true
          })
        
        }
      })

    },
    openEdit(label) {
      this.boxSelected = 'edit'
      this.label = label
    },
    openCreate() {
      if (this.boxSelected === 'create' ){
        this.boxSelected = ''
        return
      }

      this.boxSelected = 'create'
      this.label = this.getLabelDefault()
    },
  },
}
</script>

<template>
<div class="task-labels">
  <button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.label-icon 
    class="btn btn-secondary btn-block">
    {{ $t('Labels') }}
  </button>
  <b-collapse id="label-icon" @shown="getAllLabels">
    <b-card>

      <b-link  v-b-toggle.label-create>
        <font-awesome-icon :icon="['far', 'plus-square']" />
        <span>{{$t('Create a Label')}}</span>
      </b-link>

      <div v-show="boxSelected === 'edit'">
        <div class="project-label-form">
          <b-form-input v-model="label.title" size="sm" :placeholder="$t('Label Name')"></b-form-input>
          <Swatches
            v-model="label.color"
            colors="text-advanced"
            popover-to="left"
            :trigger-style="{
              width: '32px',
              height: '31px',
              borderRadius: '0',
            }"
            class="swatches-input"
          ></Swatches>
          <ButtonLoading
            type="btn-sm"
            mode="button"
            :loading="btnLoading"
            @action="editLabel(label)"
          ></ButtonLoading>
        </div>
        <div class="project-label-form-actions ">
          <a href="javascript:;" @click="boxSelected = ''">
            {{ $t('Close') }}
          </a>
          <a href="javascript:;" @click="deleteLabel(label)">
            {{ $t('Delete') }}
          </a>
        </div>
        <hr>
      </div>

      <b-collapse id="label-create">
        <div class="project-label-form mb-20-px">
          <b-form-input v-model="label.title" size="sm" :placeholder="$t('Label Name')"></b-form-input>
          <Swatches
            v-model="label.color"
            colors="text-advanced"
            popover-to="left"
            :trigger-style="{
              width: '32px',
              height: '31px',
              borderRadius: '0',
            }"
            class="swatches-input"
          ></Swatches>
          <ButtonLoading
            type="btn-sm"
            mode="button"
            :loading="btnLoading"
            @action="addLabel"
          ></ButtonLoading>
        </div>
      </b-collapse>

      <div class="label-line">

        <div v-for="labelItem in labels" :key="labelItem.id" class="dropdown-item">
          <div class="label-name d-flex align-items-center justify-content-start" @click="assignLabel(labelItem)">
            <span class="square" :style="'background:' + labelItem.color"></span>
            <span class="fw-600 ml-5-px">{{ labelItem.title }}</span>
          </div>
          <div class="action-edit" @click="openEdit(labelItem)">
            <font-awesome-icon :icon="['far', 'pencil']" style="font-size:12px" />
          </div>
        </div>

        <b-dropdown-group v-if="task.labels.length" class="dropdown-header-group-title" :header="$t('Remove Label')">
          <div v-for="taskLabel in task.labels" :key="taskLabel.id" class="dropdown-item ">
            <div class="label-name" @click="removeLabel(taskLabel)">
              <span class="square" :style="'background:' + taskLabel.color"></span>
              <span class="fw-600 ml-5-px">{{ taskLabel.title }}</span>
            </div>
            <div class="action-edit" @click="openEdit(taskLabel)">
              <font-awesome-icon :icon="['far', 'pencil']" style="font-size:12px" />
            </div>
          </div>
        </b-dropdown-group>

      </div>

    </b-card>
  </b-collapse>
</div>

</template>
