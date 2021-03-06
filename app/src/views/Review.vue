<template>
  <div class="container-fluid" style="padding-top: 80px;">
  <b-spinner label="Loading..." v-if="loading" class="float-center m-5"></b-spinner>
    <b-container fluid v-else>

      <b-row class="justify-content-md-center mt-8">
        <b-col col md="10">

          <h3>Review</h3>

          <!-- User Interface controls -->
          <b-row>
            <b-col lg="6" class="my-1">
              <b-form-group
                label="Filter"
                label-for="filter-input"
                label-cols-sm="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-input-group size="sm">
                  <b-form-input
                    id="filter-input"
                    v-model="filter"
                    type="search"
                    placeholder="Type to Search"
                  ></b-form-input>

                  <b-input-group-append>
                    <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
                  </b-input-group-append>
                </b-input-group>
              </b-form-group>
            </b-col>

            <b-col sm="5" md="6" class="my-1">
              <b-form-group
                label="Per page"
                label-for="per-page-select"
                label-cols-sm="6"
                label-cols-md="4"
                label-cols-lg="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-form-select
                  id="per-page-select"
                  v-model="perPage"
                  :options="pageOptions"
                  size="sm"
                ></b-form-select>
              </b-form-group>
            </b-col>

            <b-col sm="7" md="6" class="my-1">
              <b-pagination
                v-model="currentPage"
                :total-rows="totalRows"
                :per-page="perPage"
                align="fill"
                size="sm"
                class="my-0"
              ></b-pagination>
            </b-col>
          </b-row>

          <!-- Main table element -->
          <b-table
            :items="items"
            :fields="fields"
            :current-page="currentPage"
            :per-page="perPage"
            :filter="filter"
            :filter-included-fields="filterOn"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :sort-direction="sortDirection"
            stacked="md"
            head-variant="light"
            show-empty
            small
            fixed
            striped
            hover
            sort-icon-left
            @filtered="onFiltered"
          >

            <template #cell(actions)="row">
              <b-button 
                size="sm" 
                @click="infoReview(row.item, row.index, $event.target)" 
                class="mr-1"
                v-b-tooltip.hover.left 
                title="new review"
                >
                  <b-icon icon="pen"></b-icon>
              </b-button>
              <b-button 
                size="sm" 
                @click="infoStatus(row.item, row.index, $event.target)" 
                class="mr-1" 
                :variant="stoplights_style[row.item.category_id]"
                v-b-tooltip.hover.top 
                title="edit status"
                >
                  <b-icon icon="stoplights"></b-icon>
              </b-button>
              <b-button 
                size="sm" 
                @click="infoRemove(row.item, row.index, $event.target)" 
                class="mr-1"
                v-b-tooltip.hover.top 
                title="mark for removal"
              >
                <b-icon icon="x-circle"></b-icon>
              </b-button>
              <b-button 
                size="sm" 
                @click="infoRemove(row.item, row.index, $event.target)" 
                class="mr-1"
                v-b-tooltip.hover.right 
                title="submit entity review"
              >
                <b-icon icon="check2-circle"></b-icon>
              </b-button>
            </template>

            <template #cell(entity_id)="data">
              <b-link v-bind:href="'/Entities/' + data.item.entity_id">
                <div style="cursor:pointer">sysndd:{{ data.item.entity_id }}</div>
              </b-link>
            </template>

            <template #cell(symbol)="data">
              <b-link v-bind:href="'/Genes/' + data.item.hgnc_id"> 
                <div class="font-italic" v-b-tooltip.hover.leftbottom v-bind:title="data.item.hgnc_id">{{ data.item.symbol }}</div> 
              </b-link>
            </template>

            <template #cell(disease_ontology_name)="data">
              <b-link v-bind:href="'/Ontology/' + data.item.disease_ontology_id_version"> 
                <div v-b-tooltip.hover.leftbottom v-bind:title="data.item.disease_ontology_name + '; ' + data.item.disease_ontology_id_version">{{ truncate(data.item.disease_ontology_name, 20) }}</div> 
              </b-link>
            </template>

            <template #cell(hpo_mode_of_inheritance_term_name)="data">
                <div v-b-tooltip.hover.leftbottom v-bind:title="data.item.hpo_mode_of_inheritance_term">{{ data.item.hpo_mode_of_inheritance_term_name.replace(" inheritance", "") }}</div> 
            </template>
            
          </b-table>

        </b-col>
      </b-row>
      

      <!-- 1) Review modal -->
      <b-modal 
      :id="reviewModal.id" 
      size="xl" 
      centered 
      ok-title="Save review" 
      no-close-on-esc 
      no-close-on-backdrop 
      header-bg-variant="dark" 
      header-text-variant="light" 
      @hide="resetReviewModal" 
      @ok="handleReviewOk"
      >

        <template #modal-title>
          <h4>Entity: 
            <b-badge variant="info">
              {{ reviewModal.title }}
            </b-badge>
          </h4>
        </template>

      <b-spinner label="Loading..." v-if="loading_review_modal" class="float-center m-5"></b-spinner>
      <b-container fluid v-else>

        <form ref="form" @submit.stop.prevent="handleSubmit">

            <b-table
                :items="entity"
                :fields="entity_fields"
                small
            >

              <template #cell(symbol)="data">
                <b-link v-bind:href="'/Genes/' + data.item.hgnc_id" target="_blank"> 
                  <div class="font-italic" v-b-tooltip.hover.leftbottom v-bind:title="data.item.hgnc_id">{{ data.item.symbol }}</div> 
                </b-link>
              </template>

              <template #cell(disease_ontology_name)="data">
                <b-link v-bind:href="'/Ontology/' + data.item.disease_ontology_id_version" target="_blank"> 
                  <div class="font-italic" v-b-tooltip.hover.leftbottom v-bind:title="data.item.disease_ontology_id">{{ data.item.disease_ontology_name }}</div> 
                </b-link>
              </template>
              
            </b-table>

              <label class="mr-sm-2 font-weight-bold" for="textarea-synopsis">Synopsis</label>
                <b-badge pill id="popover-badge-help-synopsis" href="#" variant="info">
                  <b-icon icon="question-circle-fill"></b-icon>
                </b-badge>

                <b-popover target="popover-badge-help-synopsis" variant="info" triggers="focus">
                <template #title>Synopsis instructions</template>
                    Short summary for this disease entity. 
                    Please include information on: <br>
                    <strong>a)</strong> approximate number of patients described in literature, <br> 
                    <strong>b)</strong> nature of reported variants, <br>
                    <strong>c)</strong> severity of intellectual disability, <br>
                    <strong>d)</strong> further phenotypic aspects (if possible with frequencies), <br> 
                    <strong>e)</strong> any valuable further information (e.g. genotype-phenotype correlations).<br>
                </b-popover>

                <b-form-textarea
                  id="textarea-synopsis"
                  rows="3"
                  size="sm" 
                  v-model="synopsis_review"
                >
                </b-form-textarea>

              <label class="mr-sm-2 font-weight-bold" for="phenotype-select">Phenotypes</label>
                <b-badge pill id="popover-badge-help-phenotypes" href="#" variant="info">
                  <b-icon icon="question-circle-fill"></b-icon>
                </b-badge>

                <b-popover target="popover-badge-help-phenotypes" variant="info" triggers="focus">
                <template #title>Phenotypes instructions</template>
                  Add or remove associated phenotypes. 
                  Only phenotypes that occur in 20% or more of affected individuals should be included. 
                  Please also include information on severity of ID.
                </b-popover>

                <multiselect 
                  id="phenotype-select"
                  v-model="phenotypes_review"
                  tag-placeholder="Add this as new tag" 
                  placeholder="Search or add a tag" 
                  label="HPO_term" 
                  track-by="phenotype_id" 
                  :options="phenotypes_options" 
                  :multiple="true"
                  :taggable="true" 
                  @tag="addTag"
                  >
                </multiselect> 

              <label class="mr-sm-2 font-weight-bold" for="publications-select">Publications</label>
                <b-badge pill id="popover-badge-help-publications" href="#" variant="info">
                  <b-icon icon="question-circle-fill"></b-icon>
                </b-badge>

                <b-popover target="popover-badge-help-publications" variant="info" triggers="focus">
                <template #title>Publications instructions</template>
                  No complete catalogue of entity-related literature required.
                  If information in the clinical synopsis is not only based on OMIM entries, 
                  please include PMID of the article(s) used as a source for the clinical synopsis.
                </b-popover>

                <b-form-tags
                  input-id="publications-select"
                  v-model="literature_review"
                  separator=" ,;"
                  placeholder="Enter PMIDs separated by space, comma or semicolon"
                  :tag-validator="tagValidatorPMID"
                  remove-on-delete
                >
                </b-form-tags>

              <label class="mr-sm-2 font-weight-bold" for="genereviews-select">GeneReviews</label>
                <b-badge pill id="popover-badge-help-genereviews" href="#" variant="info">
                  <b-icon icon="question-circle-fill"></b-icon>
                </b-badge>

                <b-popover target="popover-badge-help-genereviews" variant="info" triggers="focus">
                <template #title>GeneReviews instructions</template>
                  Please add PMID for GeneReview article if available for this entity.
                </b-popover>

                <b-form-tags
                  input-id="genereviews-select"
                  v-model="genereviews_review"
                  separator=" ,;"
                  placeholder="Enter PMIDs separated by space, comma or semicolon"
                  :tag-validator="tagValidatorPMID"
                  remove-on-delete
                ></b-form-tags>

          <label class="mr-sm-2 font-weight-bold" for="textarea-review">Comment</label>
          <b-form-textarea
            id="textarea-review"
            rows="2"
            size="sm" 
            v-model="review_comment"
            placeholder="Additional comments to this entity relevant for the curator."
          >
          </b-form-textarea>
        </form>
      </b-container>
      </b-modal>
      <!-- 1) Review modal -->


      <!-- 2) Status modal -->
      <b-modal 
      :id="statusModal.id" 
      size="lg" 
      centered 
      ok-title="Save status" 
      no-close-on-esc 
      no-close-on-backdrop 
      header-bg-variant="dark" 
      header-text-variant="light" 
      @hide="resetReviewModal" 
      @ok="handleReviewOk"
      >
        <template #modal-title>
          <h4>Entity: 
            <b-badge variant="info">
              {{ statusModal.title }}
            </b-badge>
          </h4>
        </template>

        <form ref="form" @submit.stop.prevent="handleSubmit">
          <label class="mr-sm-2 font-weight-bold" for="status-select">Status</label>
          <b-icon 
            icon="stoplights-fill"
                :variant="stoplights_style[status_selected]"
          >
          </b-icon>
          <b-form-select 
            id="status-select" 
            class="form-control"
            :options="status_options"
            v-model="status_selected"
          >
          </b-form-select>

          <label class="mr-sm-2 font-weight-bold" for="status-textarea-comment">Comment</label>
          <b-form-textarea
            id="status-textarea-comment"
            rows="2"
            size="sm" 
            v-model="status_comment"
            placeholder="Why should this entities status be changed."
          >
          </b-form-textarea>
        </form>
      </b-modal>
      <!-- 2) Status modal -->


      <!-- 3) Removal modal -->
      <b-modal 
      :id="removeModal.id" 
      size="sm" 
      centered 
      ok-title="Suggest removal" 
      no-close-on-esc 
      no-close-on-backdrop 
      header-bg-variant="dark" 
      header-text-variant="light" 
      @hide="resetReviewModal" 
      @ok="handleReviewOk"
      >
        <template #modal-title>
          <h4>Entity: 
            <b-badge variant="info">
              {{ removeModal.title }}
            </b-badge>
          </h4>
        </template>

        <form ref="form" @submit.stop.prevent="handleSubmit">
          <div class="custom-control custom-switch">
          <input type="checkbox" class="custom-control-input" id="removeSwitch">
          <label class="custom-control-label" for="removeSwitch">Confirm removal suggestion</label>
          </div>

          <label class="mr-sm-2 font-weight-bold" for="remove-textarea-comment">Comment</label>
          <b-form-textarea
            id="remove-textarea-comment"
            rows="2"
            size="sm" 
            v-model="remove_comment"
            placeholder="Why should this entitiy be invalidated."
          >
          </b-form-textarea>
       </form>

      </b-modal>
      <!-- 3) Removal modal -->

    </b-container>

  </div>
</template>


<script>
export default {
  name: 'Review',
  data() {
        return {
          stoplights_style: {"1": "success", 2: "warning", 3: "primary", 4: "danger"},
          items: [],
          fields: [
            { key: 'entity_id', label: 'Entity', sortable: true, sortDirection: 'desc', class: 'text-left' },
            { key: 'symbol', label: 'Gene Symbol', sortable: true, class: 'text-left' },
            {
              key: 'disease_ontology_name',
              label: 'Disease',
              sortable: true,
              class: 'text-left',
              sortByFormatted: true,
              filterByFormatted: true
            },
            {
              key: 'hpo_mode_of_inheritance_term_name',
              label: 'Inheritance',
              sortable: true,
              class: 'text-left',
              sortByFormatted: true,
              filterByFormatted: true
            },
            { key: 'ndd_phenotype', label: 'NDD Association', sortable: true, class: 'text-left' },
            { key: 'actions', label: 'Actions' }
          ],
          totalRows: 1,
          currentPage: 1,
          perPage: 10,
          pageOptions: [10, 25, 50, { value: 100, text: "Show a lot" }],
          sortBy: '',
          sortDesc: false,
          sortDirection: 'asc',
          filter: null,
          filterOn: [],
          reviewModal: {
            id: 'review-modal',
            title: '',
            content: []
          },
          statusModal: {
            id: 'status-modal',
            title: '',
            content: []
          },
          removeModal: {
            id: 'removal-modal',
            title: '',
            content: []
          },
          entity: [],
          entity_fields: [
            { key: 'symbol', label: 'Gene Symbol', sortable: false, class: 'text-left' },
            {
              key: 'disease_ontology_name',
              label: 'Disease',
              sortable: false,
              class: 'text-left'
            },
            {
              key: 'hpo_mode_of_inheritance_term_name',
              label: 'Inheritance',
              sortable: false,
              class: 'text-left'
            },
            { 
              key: 'ndd_phenotype', 
              label: 'NDD Association', 
              sortable: false, 
              class: 'text-left' 
            },
            { 
              key: 'category', 
              label: 'Association Category', 
              sortable: false, 
              class: 'text-left' 
            }
          ],
          review: [{synopsis: ''}],
          review_fields: [
            { key: 'synopsis', label: 'Clinical Synopsis', class: 'text-left' },
          ],
          review_number: 0,
          synopsis_review: '',
          review_comment: '',
          status_comment: '',
          remove_comment: '',
          publications: [],
          literature_review: [],
          genereviews_review: [],
          publication_options: [],
          phenotypes_review: [],
          phenotypes_options: [],
          status_options: [],
          status_selected: 0,
          loading: true,
          loading_review_modal: true
        }
      },
      computed: {
        sortOptions() {
          // Create an options list from our fields
          return this.fields
            .filter(f => f.sortable)
            .map(f => {
              return { text: f.label, value: f.key }
            })
        }
      },
      mounted() {
        // Set the initial number of items
        this.loadEntitiesData();
        this.loadPhenotypesList();
        this.loadStatusList();
      },
      methods: {
        onFiltered(filteredItems) {
          // Trigger pagination to update the number of buttons/pages due to filtering
          this.totalRows = filteredItems.length;
          this.currentPage = 1;
        },
        resetReviewModal() {
          this.reviewModal.title = '';
          this.reviewModal.content = [];
          this.entity = [];
          this.entity_review = [];
          this.synopsis_review = '';
          this.literature_review = [];
          this.genereviews_review = [];
        },
        infoReview(item, index, button) {
          this.reviewModal.title = `sysndd:${item.entity_id}`;
          this.entity.push(item);
          this.loadEntityInfo(item.entity_id);
          this.$root.$emit('bv::show::modal', this.reviewModal.id, button);
        },
        infoStatus(item, index, button) {
          this.statusModal.title = `sysndd:${item.entity_id}`;
          this.entity.push(item);
          this.loadStatusInfo(item.entity_id);
          this.$root.$emit('bv::show::modal', this.statusModal.id, button);
        },
        infoRemove(item, index, button) {
          this.removeModal.title = `sysndd:${item.entity_id}`;
          this.entity.push(item);
          this.loadEntityInfo(item.entity_id);
          this.$root.$emit('bv::show::modal', this.removeModal.id, button);
        },
        async loadEntitiesData() {
          this.loading = true;
          let apiUrl = process.env.VUE_APP_API_URL + '/api/entities';
          try {
            let response = await this.axios.get(apiUrl);
            this.items = response.data;
            this.totalRows = response.data.length;
          } catch (e) {
            console.error(e);
          }
          this.loading = false;
        },
        async loadEntityInfo(sysndd_id) {
          this.loading_review_modal = true;

          // define API query URLs
          let apiReviewURL = process.env.VUE_APP_API_URL + '/api/entities/' + sysndd_id + '/review';
          let apiPublicationsURL = process.env.VUE_APP_API_URL + '/api/entities/' + sysndd_id + '/publications';
          let apiPhenotypesURL = process.env.VUE_APP_API_URL + '/api/entities/' + sysndd_id + '/phenotypes';

          try {
            // get API responses
            let response_review = await this.axios.get(apiReviewURL);
            let response_publications = await this.axios.get(apiPublicationsURL);
            let response_phenotypes = await this.axios.get(apiPhenotypesURL);

            // assign response data to global variables
            this.review = response_review.data;
            this.phenotypes = response_phenotypes.data;

            this.synopsis_review = this.review[this.review_number].synopsis;
            this.phenotypes_review = this.phenotypes;

            // filter the publications data into groups and assign to global variables
            let literature_filter = response_publications.data.filter(li => li.publication_type === "additional_references");
            let genereviews_filter = response_publications.data.filter(gr => gr.publication_type === "gene_review");
            Object.entries(literature_filter).forEach(([key, value]) => this.literature_review.push(value.publication_id));
            Object.entries(genereviews_filter).forEach(([key, value]) => this.genereviews_review.push(value.publication_id));

          this.loading_review_modal = false;

            } catch (e) {
            console.error(e);
            }
        },
        async loadStatusInfo(sysndd_id) {
          // define API query URLs
          let apiStatusURL = process.env.VUE_APP_API_URL + '/api/entities/' + sysndd_id + '/status';

          try {
            // get API responses
            let response_status = await this.axios.get(apiStatusURL);

            // assign response data to global variables
            this.status_selected = response_status.data[0].category_id;

            } catch (e) {
            console.error(e);
            }
        },
        async loadPhenotypesList() {
          let apiUrl = process.env.VUE_APP_API_URL + '/api/phenotypes_list';
          try {
            let response = await this.axios.get(apiUrl);
            this.phenotypes_options = response.data;
          } catch (e) {
            console.error(e);
          }
        },
        async loadStatusList() {
          let apiUrl = process.env.VUE_APP_API_URL + '/api/status_list';
          try {
            let response = await this.axios.get(apiUrl);
            //Object.entries(response.data).forEach(([key, value]) => this.status_options.push(value.category));
            this.status_options = response.data.map(item => {
              return { value: item.category_id, text: item.category };
            });

          } catch (e) {
            console.error(e);
          }
        },
        async submitReview(submission) {
          let apiUrl = process.env.VUE_APP_API_URL + '/api/entities/review?review_json=';
          try {
            let submission_json = JSON.stringify(submission);
            let response = await this.axios.post(apiUrl + submission_json);

            localStorage.setItem('submissions', submission);

          } catch (e) {
            console.error(e);
          }
        },
        handleReviewOk(bvModalEvt) {

          let review_submission = {};
          let phenotypes_submission = [];
          let modifiers_submission = [];

          Object.entries(this.phenotypes_review).forEach(([key, value]) => {
            phenotypes_submission.push(value.phenotype_id);
            modifiers_submission.push(value.modifier_id);
            }
          );

          review_submission.entity = this.entity[0].entity_id;
          review_submission.synopsis = this.synopsis_review;
          review_submission.literature = {};
          review_submission.literature.additional_references = this.literature_review;
          review_submission.literature.gene_review = this.genereviews_review;
          review_submission.phenotypes = {};
          review_submission.phenotypes.phenotype_id = phenotypes_submission;
          review_submission.phenotypes.modifier_id = modifiers_submission;

          console.log(review_submission);
          this.submitReview(review_submission);
          this.resetReviewModal();
        },
        addTag(newTag) {
            const tag = {
              phenotype_id: newTag
            }
            this.options.push(tag);
            this.value.push(tag);
          },
        tagValidatorPMID(tag) {
          // Individual PMID tag validator function
          return !isNaN(Number(tag.replace('PMID:', ''))) && tag.includes('PMID:') && tag.replace('PMID:', '').length > 4 && tag.replace('PMID:', '').length < 10;
        },
        truncate(str, n) {
          return (str.length > n) ? str.substr(0, n-1) + '...' : str;
        }
      }
  }
</script>


<style scoped>
  .btn-group-xs > .btn, .btn-xs {
    padding: .25rem .4rem;
    font-size: .875rem;
    line-height: .5;
    border-radius: .2rem;
  }
</style>