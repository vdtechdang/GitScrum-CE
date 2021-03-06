<script>
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import DatePicker from 'vue2-datepicker'
import moment from 'moment'
import ButtonLoading from '@components/utils/button-loading'
import TitleLoading from '@components/utils/title-loading'

export default {
  components: { ListUsers, DatePicker, ButtonLoading, TitleLoading},
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
      loading: true,
      btnLoading: false,

      statusAddTime: false,
      
      newTime: this.getTimeTrackingDefault(),
      period_start: null,
      period_end: null,
      alertMessage: '',
      alertStatus: false,

      timetrackings: [],
      workedHours: '',
      currentPage: 0,
      totalPages: 1,
      perPage: 15,
      loadingMore: false,
    }
  },
  created() {
    //this.setPeriod()
  },
  methods: {
    setPeriod() {
      this.period_start = null
      
      //moment(new Date()).subtract(1, 'M').format('DD-MM-YYYY')
      this.period_end = null
      //moment(new Date()).format('DD-MM-YYYY')
    },

    getTimeTrackingDefault() {
      return {
        date: null,
        startTime: null,
        endTime: null,
        comment: '',
      }
    },

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loading = true
        this.getTimetrackings()
      }
    },

    getTimetrackings() {
      this.loading = true

      Axios()
        .get(
          '/time-trackings/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid +
            '&start_date=' +
            this.period_start +
            '&end_date=' +
            this.period_end + 
            '&page=' +
            this.currentPage
        )
        .then((response) => {
          this.currentPage = response.data.current_page
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.workedHours = response.data.data.resume.worked_hours

          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            for (let i = 0; i < response.data.data.data.length; i++) {
              let timetracking = response.data.data.data[i]
              this.timetrackings.push(timetracking)
            }
          }

          this.loading = false
          
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },
    downloadExcel() {
      this.alertStatus = false
      Axios()
        .get(
          '/export-time-trackings/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&issue_slug=' +
            this.task.slug,
          { responseType: 'blob' }
        )
        .then((response) => {
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            const url = window.URL.createObjectURL(new Blob([response.data]))
            const link = document.createElement('a')
            link.href = url
            link.setAttribute('download', this.task.title + '.xlsx')
            document.body.appendChild(link)
            link.click()
          }
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },

    addTime() {
      this.alertStatus = false
      this.btnLoading = true
      if (!this.newTime.date) {
        this.btnLoading = false
        return
      }

      let d0 = new Date(this.newTime.date)
      let startDate =
        d0.getFullYear() + '-' + ('0' + (d0.getMonth() + 1)).slice(-2) + '-' + ('0' + d0.getDate()).slice(-2)
      let endDate = startDate

      let startDateTime = new Date(startDate + ' ' + this.newTime.startTime + ':00').toUTCString()
      let endDateTime = new Date(endDate + ' ' + this.newTime.endTime + ':00').toUTCString()

      Axios()
        .post('/time-trackings/?company_slug=' + this.task.company.slug + '&project_slug=' + this.task.project.slug, {
          task_uuid: this.task.uuid,
          start: moment.utc(startDateTime).format('YYYY-MM-DD HH:mm:ss'),
          end: moment.utc(endDateTime).format('YYYY-MM-DD HH:mm:ss'),
          comment: this.newTime.comment,
        })
        .then((response) => {
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.timetrackings.push(response.data.data)
            this.newTime = this.getTimeTrackingDefault()
          }
          this.btnLoading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
          this.btnLoading = false
        })
    },

    removeTime(id) {

      this.$bvModal.msgBoxConfirm(this.$t('Are you sure you want to delete this time tracking?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){

          Axios()
          .delete(
          '/time-trackings/' +
            id +
            '/' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
          )
          .then((response) => {
            let found = this.timetrackings.find((t) => t.time.id === id)
            this.timetrackings.splice(this.timetrackings.indexOf(found), 1)
          })
          .catch((error) => {
            this.alertMessage = 'Error. ' + error.response.data.message
            this.alertStatus = true
          })

        }
      })
      
    },
  },
}
</script>
<template>
  <div>
    <button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.timetracking-icon
    class="btn btn-secondary btn-block">
      {{ $t('Time Tracking') }}
    </button>
    <b-collapse id="timetracking-icon">
      <b-card>
        <div class="d-flex justify-content-between mb-2">
          <div class="badge badge-light font-weight-bold">{{ $t('Log') + ': ' + workedHours }}</div>
          <div>
            <b-link v-b-toggle.timetracking-form class="badge badge-light">
              <font-awesome-icon :icon="['far', 'plus']" />
              {{$t('Add Time')}}
            </b-link>
            <b-link  class="badge badge-light mr-0" @click="downloadExcel">
              <font-awesome-icon :icon="['far', 'cloud-download']" />
              {{$t('Download')}}
            </b-link>
          </div>
        </div>
        <b-collapse id="timetracking-form" class="mb-2">
          <DatePicker
            v-model="newTime.date"
            lang="en"
            format="YYYY-MM-DD"
            confirm
            style="width: 100% !important;"
            :not-after="new Date()"></DatePicker>
          <div class="d-flex mt-5px mb-5px">
            <DatePicker
              v-model="newTime.startTime"
              lang="en"
              value-type="format"
              format="HH:mm"
              type="time"
              placeholder="Start Time"
              class="time-picker mr-5-px"
              :minute-step="5"
              confirm></DatePicker>
            <DatePicker
              v-model="newTime.endTime"
              lang="en"
              value-type="format"
              format="HH:mm"
              type="time"
              placeholder="End Time"
              class="time-picker ml-5-px"
              :minute-step="5"
              confirm></DatePicker>
          </div>
          <b-form-input 
          v-model="newTime.comment" 
          :placeholder="$t('Write a comment')" 
          size="sm" class="mb-5px"></b-form-input>
          <div class="d-flex justify-content-end">
            <ButtonLoading
            type="btn-md"
            mode="button"
            :loading="btnLoading"
            :title="$t('Add Time Log')"
            :title-loading="$t('Adding')"
            @action="addTime"></ButtonLoading>
          </div>
        </b-collapse>
        <div 
          v-infinite-scroll="loadMore" 
          infinite-scroll-disabled="busy" 
          infinite-scroll-distance="200"
          style="max-height:316px; overflow-x:auto;">

          <b-card v-for="(item, index) in timetrackings" :key="index" class="mb-1">
            <div v-if="item.comment" 
              :title="item.comment" 
              :alt="item.comment" class="small txt-muted">{{ item.comment | truncate(95) }}</div>
            <b-link class="small txt-primary" v-text="item.user.name"></b-link>
            <div class="small"> 
              {{ item.time.start.timezone }} {{ $t('to')}}
              <span v-if="item.time.end.timezone" v-text="item.time.end.timezone" />
              <span v-if="!item.time.end.timezone">{{ $t('current time') }}</span>
            </div>
            <div class="d-flex justify-content-between">
              <div v-if="item.time.end.timezone" class="small font-weight-bold">
                {{ $t('Time Spent') }}: {{ item.time.total }}
              </div>
              <div
                v-if="authorize('tasks', 'delete')"
                class="card-delete"
                @click="removeTime(item.time.id)">
                {{ $t('Delete') }}
              </div>
            </div>
          </b-card>
        </div>
        <b-link 
        :to="{
        name: 'projects.addons.time-tracking',
        params: {
          companySlug: task.company.slug,
          projectSlug: task.project.slug } }" 
        class="small txt-primary" 
        v-text="$t('Go to Project Time Tracking')" />
      </b-card>
    </b-collapse>
  </div>
</template>