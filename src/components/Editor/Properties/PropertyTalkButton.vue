<template>
	<button v-if="writableEvent"
		class="talk-button"
		:disabled="isCreateTalkRoomButtonDisabled"
		@click="createTalkRoom">
		Add video call link
	</button>
</template>

<script>
import { createTalkRoom, doesDescriptionContainTalkLink } from '../../../services/talkService.js'
import { NEW_LINE } from '../../../utils.js'

export default {
	name: 'PropertyTalkButton',
	props: {
		calendarObjectInstance: {
			type: Object,
			required: true,
		},
		isReadOnly: {
			type: Boolean,
			required: true,
		},
		isSlot: {
			type: Boolean,
			default: false,
		},
	},
	computed: {
		isCreateTalkRoomButtonDisabled() {
			if (this.creatingTalkRoom) {
				return true
			}

			if (doesDescriptionContainTalkLink(this.calendarObjectInstance.description)) {
				return true
			}

			return false
		},
		writableEvent() {
			return !this.isSlot && !this.isReadOnly
		},
	},
	methods: {
		async createTalkRoom() {
			try {
				this.creatingTalkRoom = true
				const url = await createTalkRoom(this.calendarObjectInstance.title)

				let newDescription
				if (!this.calendarObjectInstance.description) {
					newDescription = url
				} else {
					newDescription = this.calendarObjectInstance.description + NEW_LINE + url
				}

				this.$store.commit('changeDescription', {
					calendarObjectInstance: this.calendarObjectInstance,
					description: newDescription,
				})
			} catch (error) {
				alert(this.$t('calendar', 'Error creating Talk room'))
			} finally {
				this.creatingTalkRoom = false
			}
		},
	},
}
</script>

<style scoped>
.talk-button {
	margin: 3px 0;
	background-color: #fff !important;
	border: 2px solid var(--color-border-dark);
	border-radius: 7px;
}
</style>
