<template>
  <div class="main-layout-box">
    <m-secondary-menu :type="'security'"></m-secondary-menu>
    <template>
      <m-list-construction :title="$t('Queue manage')">
        <template slot="conditions">
          <m-conditions @on-conditions="_onConditions">
            <template slot="button-group">
              <x-button type="ghost" size="small" @click="_create('')">{{$t('Create queue')}}</x-button>
            </template>
          </m-conditions>
        </template>
        <template slot="content">
          <template v-if="queueList.length">
            <m-list :queue-list="queueList" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize"></m-list>
            <div class="page-box">
              <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
            </div>
          </template>
          <template v-if="!queueList.length">
            <m-no-data></m-no-data>
          </template>
          <m-spin :is-spin="isLoading"></m-spin>
        </template>
      </m-list-construction>
    </template>
  </div>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mSpin from '@/module/components/spin/spin'
  import mCreateQueue from './_source/createQueue'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mSecondaryMenu from '@/module/components/secondaryMenu/secondaryMenu'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'queue-index',
    data () {
      return {
        total: null,
        isLoading: true,
        queueList: [],
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: ''
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('security', ['getQueueListP']),
      /**
       * Query
       */
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },
      _page (val) {
        this.searchParams.pageNo = val
      },
      _create (item) {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mCreateQueue, {
              on: {
                onUpdate () {
                  self._debounceGET('false')
                  modal.remove()
                },
                close () {
                  modal.remove()
                }
              },
              props: {
                item: item
              }
            })
          }
        })
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getQueueListP(this.searchParams).then(res => {
          this.queueList = []
          this.queueList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {

    },
    components: { mSecondaryMenu, mList, mListConstruction, mConditions, mSpin, mNoData }
  }
</script>
