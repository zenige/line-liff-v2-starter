<template>
  <div class="container">
    <Loader v-if="!isLoading" />
    <div v-else class="hc_navbar">
      <div class="container fitscreen" style="padding: 2rem 0">
        <div class="row d-flex align-items-center">
          <div class="col-8 col-md-4">
            <div class="border_search_gray form-group-feedback-right">
              <div class="input-group">
                <span class="input-group-append"
                  ><span
                    class="
                      input-group-text input-group-text-search-border
                      rounded-left
                    "
                    ><i class="icon-search4 txt_grey mr-2"></i></span
                ></span>
                <input
                  v-model="filter"
                  type="search"
                  class="
                    form-control
                    rounded-right
                    form-control-search-border
                    h2dot5
                  "
                  style="margin-right: 1rem"
                  placeholder="Search for a group...."
                />
                <div class="form-control-feedback" @click="filter = ''">
                  <i class="icon-cross3 txt_grey" style="height: 22px"></i>
                </div>
              </div>
            </div>
          </div>
          <div class="col-4 col-md-8 text-right">
            <button
              @click="openCreateGroupModal()"
              class="hcb-btn btn btn_hcb_blue btn-block"
            >
              Create group
            </button>
            <button
              @click="openTrainModelModal()"
              class="hcb-btn btn btn_hcb_green btn-block"
            >
              Train model
            </button>
          </div>
        </div>
        <div class="col-md-12" style="padding-top: 2rem; padding-bottom: 1rem">
          <div class="row d-flex align-items-center">
            <div class="txt_hc_head_total">
              Total group
              <span class="txt_vla_grey">({{ groupData.length }})</span>
            </div>
          </div>
        </div>
        <div class="col-md-12">
          <div class="row d-flex align-items-center justify-content-center">
            <b-table
              responsive
              id="Group-table"
              :items="groupData"
              :per-page="0"
              :current-page="currentPage"
              :fields="fields"
              :filter="filter"
              :tbody-tr-class="selectedRowClass"
            >
              <template #cell(group)="data">
                <div v-if="data.item.editable === false">
                  <div>
                    {{ data.item.group }}
                  </div>
                </div>
                <div v-if="data.item.editable === true" style="width: 35%">
                  <b-form-input
                    name="groupName"
                    autofocus
                    v-model="changedGroupData[data.index]"
                  />
                </div>
              </template>
              <template #head(action)>
                <div class="d-flex justify-content-center">Action</div>
              </template>
              <template #cell(action)="data">
                <div
                  v-if="data.item.editable === false"
                  class="d-flex justify-content-center"
                >
                  <div class="header-elements">
                    <div class="list-icons">
                      <div class="breadcrumb-elements-item dropdown p-0 mx-1">
                        <b-dropdown right variant="link" no-caret>
                          <template v-slot:button-content>
                            <i class="icon-more2 txt_grey"></i>
                          </template>
                          <span @click="editGroup(data)" class="dropdown-item">
                            Rename
                          </span>
                          <NuxtLink
                            :to="
                              localePath(
                                '/chatbot-training/group-management/group/' +
                                  data.item.id
                              )
                            "
                            class="dropdown-item"
                          >
                            Edit group
                          </NuxtLink>
                          <div class="dropdown-divider"></div>
                          <span
                            name="deleteGroup"
                            class="dropdown-item txt_red"
                            @click="openDeleteGroupModal(data)"
                          >
                            Delete
                          </span>
                        </b-dropdown>
                      </div>
                    </div>
                  </div>
                </div>
                <div
                  class="row d-flex justify-content-center align-items-center"
                  v-if="data.item.editable === true"
                >
                  <button
                    @click="saveGroup(data)"
                    class="hcb-btn-light btn btn_hcb_green_light mr-2 btn-block"
                  >
                    Save
                  </button>
                  <div @click="cancleEditGroup(data)" class="txt_grey_cancel">
                    Cancel
                  </div>
                </div>
              </template>
            </b-table>
            <div style="margin-top: 0.5rem">
              <b-pagination
                v-model="currentPage"
                :total-rows="totalGroup"
                :per-page="perPage"
                aria-controls="Group-table"
                first-number
                last-number
                align="center"
                class="myPaginattion"
              ></b-pagination>
            </div>
          </div>
        </div>

        <TrainModelModal
          :isOpen="isShowTrainModelModal"
          :onCancel="closeTrainModelModal"
          :onTrain="TrainModel"
        ></TrainModelModal>
        <DeleteGroupModal
          :isOpen="isShowDeleteGroupModal"
          :onCancel="closeDeleteGroupModal"
          :onDelete="onDeleteGroup"
        ></DeleteGroupModal>
        <CreateGroupModal
          :isOpen="isShowCreateGroupModal"
          :onCancel="closeCreateGroupModal"
          @getGroupData="getGroupData"
        ></CreateGroupModal>
      </div>
    </div>
  </div>
</template>

<script>
const english = /^[A-Za-z0-9^*()_+=[/\]{}|\\,.?: -]*$/

export default {
  components: {
    CreateGroupModal: () => import('~/components/modals/CreateGroupModal.vue'),
    TrainModelModal: () => import('~/components/modals/TrainModelModal.vue'),
    DeleteGroupModal: () => import('~/components/modals/DeleteGroupModal.vue'),
    Loader: () => import('~/components/Loader.vue'),
  },
  props: {},
  data() {
    return {
      isLoading: false,
      isShowCreateGroupModal: false,
      isShowTrainModelModal: false,
      isShowDeleteGroupModal: false,
      edit: false,
      selectAll: false,
      selectedRow: {},
      filter: '',
      perPage: 10,
      currentPage: 1,
      deleteSelected: {},
      totalGroup: 0,
      newGroupName: '',
      oldGroupName: '',
      fields: [
        {
          key: 'group',
          label: 'Group name',
          sortable: true,
          thClass: 'groupthGroup-Class',
          tdClass: 'grouptdGroup-Class',
        },
        {
          key: 'action',
          label: 'Action',
          sortable: false,
          thClass: 'groupthAction-Class',
          tdClass: 'grouptdAction-Class',
        },
      ],
      groupData: [],
      changedGroupData: [],
    }
  },
  watch: {
    selectAll(value) {
      this.groupData.map(function (item) {
        item.selected = value
        // return item
      })
    },
    currentPage(value) {
      this.getGroupData(value, this.perPage, 'question')
    },
  },
  async mounted() {
    await this.getGroupData()
    this.isLoading = true
  },
  computed: {},
  methods: {
    openDeleteGroupModal(data) {
      this.isShowDeleteGroupModal = true
      this.deleteSelected = data
    },
    async onDeleteGroup() {
      await this.$axios.delete('group/' + this.deleteSelected.item.id)
      this.groupData = this.groupData.filter(
        (e) => e.id != this.deleteSelected.item.id
      )
      this.deleteSelected = {}
    },
    closeDeleteGroupModal() {
      this.isShowDeleteGroupModal = false
    },
    async TrainModel() {
      await this.$axios.get('logic/updateLogic')
      await this.$axios.post(
        'http://localhost:500/trainModel/createModelRF',
        {}
      )
      this.$bvToast.toast('Trained successfully', {
        variant: 'success',
        toaster: 'b-toaster-bottom-left',
        noCloseButton: true,
      })
    },
    selectedRowClass(item) {
      if (item.selected === true) return 'row-selected'
    },
    openTrainModelModal() {
      this.isShowTrainModelModal = true
    },
    closeCreateGroupModal() {
      this.selectAll = false
      this.isShowCreateGroupModal = false
    },
    openCreateGroupModal() {
      this.isShowCreateGroupModal = true
    },
    closeTrainModelModal() {
      this.isShowTrainModelModal = false
    },
    editGroup(data) {
      this.changedGroupData[data.index] = data.item.group
      data.item.editable = true
    },
    cancleEditGroup(data) {
      data.item.editable = false
    },
    async saveGroup(data) {
      data.item.group = this.changedGroupData[data.index]
      data.item.editable = false
      if (data.item.id === data.item.group) {
        await this.getGroupData()
      } else {
        if (english.test(data.item.id)) {
          await this.$axios.patch('group', {
            id: data.item.id,
            Name: data.item.group,
          })
          await this.getGroupData()
        } else {
          this.$bvToast.toast('Please fill in matching text.', {
            variant: 'danger',
            toaster: 'b-toaster-bottom-left',
            noCloseButton: true,
          })
        }
      }
    },
    async getGroupData() {
      let { data } = await this.$axios.get('group')
      this.groupData = data.map((item) => {
        item.data = item.data.replaceAll("'", '"')
        let group = JSON.parse(item.data)
        return {
          group: group.group,
          id: item.id,
          editable: false,
        }
      })
      this.totalGroup = this.groupData.lenght
      if (this.groupData.length === 0) {
        this.totalGroup = 0
      }
      // console.log('all group data: ', this.groupData)
    },
  },
}
</script>

<style lang="scss">
.groupthGroup-Class,
.grouptdGroup-Class {
  width: 80%;
}
.groupthAction-Class,
.grouptdAction-Class {
  width: 20%;
}
</style>
