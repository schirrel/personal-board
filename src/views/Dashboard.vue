<template>
  <div>
    <vs-row vs-justify="center">
      <img :src="Logo" width="200" height="80" alt="Personal Board">
      <vs-col  vs-type="flex" vs-align="center" vs-justify="center" vs-w="12">
        <vs-col vs-type="flex" vs-justify="flex-start" vs-align="center" vs-w="6">
          <vs-avatar class="ml-10" size="large" src="https://avatars2.githubusercontent.com/u/31676496?s=460&v=4"/>
          <h1 class="ml-10">{{ $t('welcome-user')}} {{ username }}</h1>
        </vs-col>
        <vs-col vs-type="flex" vs-justify="space-between" vs-align="center" vs-w="6">
          <vs-col vs-type="flex" vs-align="flex-end" vs-w="2"></vs-col>
          <vs-col vs-type="flex" vs-align="flex-end" vs-w="2"></vs-col>
          <vs-col vs-type="flex" vs-justify="center" vs-align="flex-end" vs-w="2">
            <vs-button style="border-radius: 50%" color="success" type="relief" @click="isActive = true" icon="add"></vs-button>
          </vs-col>
        </vs-col>
      </vs-col>
      <vs-divider position="center">
        {{$t('my-board')}}
      </vs-divider>
      <draggable class="vs-row" v-model="reactiveBoard" style="justify-content: flex-start; display: flex; width: 100%;">
        <vs-col vs-type="flex" vs-justify="flex-start" vs-align="flex-start" vs-lg="3" vs-sm="5" vs-xs="12"  vs-w="3" :key="board.id" v-for="board in reactiveBoard">
          <vs-card actionable>
            <div slot="header" @click="goToBoard(board)">
              <vs-row vs-justify="center">
                <vs-col vs-type="flex" vs-w="12">
                  <vs-col vs-type="flex" vs-w="6">
                    <h3>
                      {{board.title}}
                    </h3>
                  </vs-col>
                  <vs-col vs-type="flex" vs-justify="flex-end" vs-align="flex-end" vs-w="6">
                    <vs-button type="flat" color="rgb(255,223,0)" icon="star_border"></vs-button>
                  </vs-col>
                </vs-col>
              </vs-row>
            </div>
            <div>
              <span>{{board.description}}</span>
            </div>
            <div slot="footer">
              <vs-row vs-justify="flex-end">
                <vs-button class="ml-5" type="relief" color="danger" icon="delete" @click.prevent="exclude(board.id)"></vs-button>
                <vs-button class="ml-5" type="relief" color="primary" icon="edit" @click.prevent="openBoard(board)"></vs-button>
              </vs-row>
            </div>
          </vs-card>
        </vs-col>
      </draggable>
      <vs-prompt
        @cancel="clearDialog"
        @accept="chooseAction"
        :title="!isBoardSelected ? 'New Board' : 'Edit Board'"
        :is-valid="validField"
        :accept-text="!isBoardSelected ? $t('create') : $t('edit')"
        :cancel-text="$t('cancel')"
        :active.sync="isActive"
        :color="'success'"
        class="con-vs-dialog">
          <div class="con-exemple-prompt">
          <span v-html="$t('board-dialog')"></span>
            <vs-input :placeholder="$t('board-name-placeholder')" v-model="title"/>
            <vs-textarea :label="$t('board-description-placeholder')" v-model="description" />

            <vs-alert :active="!validField" color="danger" icon="new_releases" >
              {{$t('dialog-invalid')}}
            </vs-alert>
          </div>
      </vs-prompt>
    </vs-row>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import draggable from 'vuedraggable'
import Logo from '@/assets/logo/personalboard-horiz.png'

export default {
  name: 'Dashboard',
  components: {
    draggable
  },
  computed: {
    ...mapState('Global', ['username', 'boards']),
    validField () {
      return (this.title.length > 0 && this.description.length > 0)
    },
    reactiveBoard: {
      set (value) {
        this.tempBoard = [...value]
        this.saveBoards(this.tempBoard)
      },
      get () {
        return this.tempBoard
      }
    }
  },
  data: () => ({
    isActive: false,
    title: '',
    description: '',
    isBoardSelected: false,
    boardSelected: {},
    tempBoard: [],
    Logo: Logo
  }),
  mounted() {
    this.tempBoard = [...this.boards]
  },
  methods:{
    ...mapActions('Global', ['createBoard', 'deleteBoard', 'editBoard', 'saveBoards']),
    create () {
      const payload = {
        title: this.title,
        description: this.description
      }
      this.createBoard(payload)
    },
    clearDialog(){
      this.title = ''
      this.description = ''
      this.isBoardSelected = false
    },
    openBoard ( board ) {
      this.isBoardSelected = true
      this.boardSelected = {...board}
      this.isActive = true
      this.title = board.title
      this.description = board.description
      
    },
    edit ( board ) {
      const payload = {
        id: board.id,
        title: this.title,
        description: this.description,
        starred: board.starred,
        lists: board.lists
      }
      this.editBoard(payload)
    },
    exclude ( id ) {
      const payload = {
        id: id
      }
      this.deleteBoard(payload)
      this.reactiveBoard = [...this.boards]
    },
    goToBoard ( board ) {
      this.$router.push({ name: 'board', params: { board: board}})
    },
    chooseAction () {
      if ( !this.isBoardSelected ) this.create()
      else this.edit(this.boardSelected)
      this.$vs.notify({
        color:'primary',
        title: !this.isBoardSelected ? 'New Board Created' : 'Board edited',
        text: `Board title: ${this.title}`
      })
      this.reactiveBoard = [...this.boards]
      this.clearDialog()
    }
  }
}
</script>

<style scoped>
.con-vs-card {
  margin-right: 10px;
}

.ml-10 {
  margin-left: 10px;
}

.ml-5 {
  margin-left: 5px;
}

.con-exemple-prompt {
  padding: 10px;
  padding-bottom: 0;
}

.con-exemple-prompt .vs-input {
  width: 100%;
  margin-top: 10px;
}

.vs-con-textarea {
  width: 100%;
  margin-top: 10px;
}
</style>


