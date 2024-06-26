<style scoped lang="scss">

</style>

<template lang="pug">
  .root
    headers(:title="this.$t('.title')")

    .container.tool.items.u-cf(v-if="items")
      .card
        span
          | {{ this.$t('.list.title') }}

        router-link.button.button-primary.u-pull-right.router-link(:to="{ name: 'newItem' }" v-if="$ability.canManage('Item')")
          | {{ this.$t('.button.new') }}

      filter-bar(:placeholder="$t('.search.placeholder')", @filter="filter")

      table-list
        table-header(resourceClass='Item')
          th.sort-link(@click="sort('title')")
            | {{ $t('models.item.one') }}

            i.fa(:class="sortClass" v-if="this.sortParam === 'title'")

          th
            | {{ $t('models.item.attributes.classificationId') }}

          th
            | {{ $t('models.item.attributes.unit') }}

        tbody(v-if="itemsCount")
          tr(v-for="item in items")
            td
              router-link.router-link(:to="{ name: 'item', params: { id: item.id } }")
                | {{ item.title }}
            td
              router-link.router-link(:to="{ name: 'item', params: { id: item.id } }")
                | {{ item.classification_name.split(" / ").map((e,i)=>$t("classification.label."+e)).join(" / ")  }}
            td
              router-link.router-link(:to="{ name: 'item', params: { id: item.id } }")
                | {{ item.unit_name }}

            template(v-if="$ability.canManage('Item')")
              td
                router-link.button.router-link(:to="{ name: 'editItem', params: { id: item.id } }")
                  | {{ $t('btn.edit') }}

              td-destroy(:id="item.id", :title="$t('.destroy.title')", :body="$t('.destroy.body')", @success="destroy(item.id)")

        tbody(v-else)
          tr
            td.text-center(colspan='5')
              | {{ $t('.empty') }}

      .row
        .twelve.columns
          paginator(
            v-model="page",
            :first="firstPageLink",
            :prev="prevPageLink",
            :next="nextPageLink",
            :last="lastPageLink",
            v-if="itemsCount"
          )

</template>

<script>
  import parseLinkHeaders from 'parse-link-header'

  export default {
    name: 'items',
    data () {
      return {
        i18nScope: 'items.index',

        // resource
        items: null,
        itemsCount: null,
        params: {},

        //search
        search: '',
        searching: false,

        // sort
        sortParam: 'title',
        orderParam: 'asc',

        // pagination
        page:          1,
        firstPageLink: '',
        prevPageLink:  '',
        nextPageLink:  '',
        lastPageLink:  '',
        totalPages:    1,
      }
    },

    computed: {
      fetchParams() {
        return this.params
      },

      sortClass() {
        return (this.orderParam === 'asc') ? 'fa-caret-down' : 'fa-caret-up'
      }
    },

    methods: {
      fetch() {
        let params = this.params

        return this.$http.get('/administrator/items', { params })
          .then((response) => {
            this.items = response.data
            this.itemsCount = this.items.length

            this.updatePagination(response)

          }).catch((_err) => {
            this.error = _err
            console.error(_err)
          })
      },

      updatePagination(aResponse) {
        this.page = aResponse.headers['x-page']
        this.totalPages = aResponse.headers['x-total']
        let links = parseLinkHeaders(aResponse.headers.link) || {}

        this.firstPageLink = _.dig(links, 'first', 'page')
        this.prevPageLink = _.dig(links, 'prev', 'page')
        this.nextPageLink = _.dig(links, 'next', 'page')
        this.lastPageLink = _.dig(links, 'last', 'page')
      },

      filter(aSearch) {
        this.search = aSearch
        this.fetchSearch()
      },

      fetchSearch() {
        this.params = { search: this.search }

        this.updateRoute()
      },

      updateRoute() {
        let data = Object.assign({}, this.$route.query, this.params);

        this.$router.push({ name: 'items', query: data });
      },

      sort(aSort) {
        if(aSort === this.sortParam) {
          this.orderParam = this.orderParam==='asc' ? 'desc' : 'asc';
        }

        this.sortParam = aSort;

        let sortParams = { sort_column: this.sortParam, sort_direction: this.orderParam }

        this.params = Object.assign({}, this.params, sortParams);

        this.updateRoute()
      },

      destroy(id) {
        return this.$http.delete('/administrator/items/' + id)
          .then((_response) => {
            let idx = this.items.findIndex(p => p.id === id)
            this.items.splice(idx, 1)
            this.$notifications.info(this.$t('.notifications.destroy.success', { id }))
          })
          .catch((_err) => {
            this.$notifications.error(this.$t('.notifications.destroy.failure', { id }))
          })
      },

      init() {
        this.params = this.$route.query
      }
    },

    created: function () {
      this.init();
    },

    watch: {
      fetchParams() {
        this.fetch()
      },

      page() {
        this.params = Object.assign({}, this.params, { page: this.page });
        this.updateRoute()
      }
    }
  }

</script>
