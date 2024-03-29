<template>
    <section>
        <section>
            <template-header
                v-if="board && board.isTemplate"
                :board="board"
                @duplicateTemplate="addTemplateBoard"
            ></template-header>
            <board-header
                v-if="board"
                :board="board"
                @titleChange="updateBoardTitle"
                @starredChange="isStarredBoard"
                @changeBgcColor="changeBoardBgcColor"
                @changeBoardBgc="changeBoardPhoto"
                @makeMember="inviteUser"
                @removeInvitedUser="removeMember"
            />
        </section>
        <section
            v-if="board"
            class="board-details-main"
            :style="{ 'backgroundColor': board.background }"
        >
            <group-list
                @saveCopy="saveCopyToBoard"
                @boardModified="updateBoard"
                @removeGroup="groupRemoveFromBoard"
                @openCardDetails="openCardDetailsModal"
                @groupUpdated="updateGroup"
                :groups="board.groups"
                :board="board"
                :newGroup="newGroup"
                @addGroup="addNewGroup"
                @groupDnd="updateBoardDnd"
            ></group-list>
        </section>
        <section v-if="isCardOpen" class="dialog-container card-details-scroll">
            <card-details
                ref="cardDetailsModal"
                class="modal"
                @groupUpdate="updateGroup"
                @deleteCardFromGroup="deleteCardFromGroup"
                @boardModified="updateBoard"
                @cardModified="updateCard"
                @closeDialog="closeDiag"
                @saveCopy="saveCopyToBoard"
                :board="board"
                :card="cardToShow"
                :group="groupToShow"
            ></card-details>
        </section>
    </section>
</template>

<script>
import { boardService } from "../services/board-service.js"
import { socketService } from "../services/socket.service.js"
import groupList from "../components/group-list.vue"
import boardHeader from "../components/board-header.vue"
import templateHeader from "../components/template-header.vue"
import cardDetails from "../components/card-details.vue"

export default {
    components: {
        groupList,
        boardHeader,
        cardDetails,
        socketService,
        templateHeader
    },
    data() {
        return {
            isCardOpen: false,
            selectedCardIdx: null,
            selectedCardGroupIdx: null,
            boardToEdit: null,
            groupToEdit: null,
            newGroup: boardService.getEmptyGroup(),
            newActivity: boardService.getEmptyActivity(),
        }
    },
    created() {
    },
    mounted() {
        socketService.on('board-changed', this.socketBoardUpdate)
    },
    methods: {
        socketBoardUpdate(board) {
            this.$store.commit({ type: 'setBoard', board })
        },
        groupRemoveFromBoard(groupId) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.newActivity = boardService.getEmptyActivity()
            if (this.loggedinUser) this.newActivity.byMember = this.loggedinUser
            const idx = this.boardToEdit.groups.findIndex((group) => group.id === groupId)
            this.saveActivity('removed list ' + this.boardToEdit.groups[idx].title)
            this.boardToEdit.groups.splice(idx, 1)
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        addNewGroup(newGroup) {
            if (!newGroup.title) return
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.saveActivity('added ' + newGroup.title + '  to this board')
            this.boardToEdit.groups.push(newGroup)
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
            this.newGroup = boardService.getEmptyGroup()
            this.show = false;
        },
        close() {
            this.show = false;
            this.newGroup.title = ""
        },
        updateCard({ card, group }) {
            this.groupToEdit = JSON.parse(JSON.stringify(group))
            const cardToEditIdx = this.groupToEdit.cards.findIndex(cardToFind => cardToFind.id === card.id)
            this.groupToEdit.cards[cardToEditIdx] = card
            this.updateGroup(this.groupToEdit)
        },
        deleteCardFromGroup({ card, group }) {
            this.groupToEdit = JSON.parse(JSON.stringify(group))
            const cardToEditIdx = this.groupToEdit.cards.findIndex(cardToFind => cardToFind.id === card.id)
            this.groupToEdit.cards.splice(cardToEditIdx, 1)
            this.updateGroup(this.groupToEdit)
        },
        updateGroup(editedGroup) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            const groupIdx = this.boardToEdit.groups.findIndex(group => group.id === editedGroup.id)
            this.boardToEdit.groups[groupIdx] = editedGroup
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        updateBoard({ key, val }) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.boardToEdit[key] = val
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        openCardDetailsModal(info) {
            this.selectedCardIdx = this.board.groups.find(group => group.id === info.group.id).cards.findIndex(card => card.id === info.card.id)
            this.selectedCardGroupIdx = this.board.groups.findIndex(group => group.id === info.group.id)
            this.isCardOpen = true
        },
        closeDiag() {
            this.isCardOpen = false
        },
        saveCopyToBoard(copy) {
            this.groupToEdit = JSON.parse(JSON.stringify(copy.posCopy.group))
            this.groupToEdit.cards.splice(copy.posCopy.position - 1, 0, copy.cardCopy)
            this.updateGroup(this.groupToEdit)
            socketService.emit('update board', copy)
        },
        updateBoardDnd(newBoard) {
            this.$store.dispatch({ type: 'saveBoard', board: newBoard })
        },
        updateBoardTitle(title) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.saveActivity('renamed this board from ' + '(' + this.board.title + ')')
            this.boardToEdit.title = title
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        isStarredBoard(isStarred) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.boardToEdit.isStarred = isStarred
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        changeBoardBgcColor(color) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))

            this.saveActivity('changed the background of this board')

            this.boardToEdit.background = color
            this.boardToEdit.backgroundPhoto = ''
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        changeBoardPhoto(photo) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.saveActivity('changed the background of this board')
            this.boardToEdit.background = ''
            this.boardToEdit.backgroundPhoto = photo.full
            this.boardToEdit.backgroundThumb = photo.thumb
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        saveActivity(txt) {
            if (this.loggedinUser) this.newActivity.byMember = this.loggedinUser
            this.newActivity.txt = txt
            this.boardToEdit.activities.unshift(this.newActivity)
            this.newActivity = boardService.getEmptyActivity()
        },
        inviteUser(user) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            this.boardToEdit.members.push(user)
            this.saveActivity(' added ' + user.fullname + ' to this board')
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        removeMember(user) {
            this.boardToEdit = JSON.parse(JSON.stringify(this.board))
            const idx = this.boardToEdit.members.findIndex((member) => member._id === user._id)
            this.boardToEdit.members.splice(idx, 1)
            this.saveActivity(' removed ' + user.fullname + ' from this board')
            this.$store.dispatch({ type: 'saveBoard', board: this.boardToEdit })
        },
        addTemplateBoard(board) {
            if (this.loggedinUser) {
                board.createdBy = this.loggedinUser
                board.members.push(this.loggedinUser)
                this.newActivity.byMember = this.loggedinUser
            }
            board.isTemplate = false
            this.newActivity.txt = 'created this board'
            board.activities.push(this.newActivity)
             this.newActivity = boardService.getEmptyActivity()
            this.$store.dispatch({ type: 'saveBoard', board: board })
        }
    },
    computed: {
        board() {
            return this.$store.getters.board
        },
        cardToShow() {
            return this.board.groups[this.selectedCardGroupIdx].cards[this.selectedCardIdx]
        },
        groupToShow() {
            return this.board.groups[this.selectedCardGroupIdx]
        },
        loggedinUser() {
            return this.$store.getters.loggedinUser
        },

    },
    watch: {
        "$route.params.boardId": {
            async handler(newId) {
                if (newId) {
                    await this.$store.dispatch({ type: 'loadBoardById', newId })

                    if (this.loggedinUser) {
                        socketService.emit('watch board', newId)
                        socketService.emit('set-user-socket', this.loggedinUser._id)
                    }
                }
            },
            immediate: true
        },
    }
}

</script>

<style>
.modal {
    border-radius: 2px;
    box-shadow: 0 0 1em rgb(0, 0, 0/0.3);
}

.modal::backdrop {
    background: black;
    opacity: 0.3;
}
</style>