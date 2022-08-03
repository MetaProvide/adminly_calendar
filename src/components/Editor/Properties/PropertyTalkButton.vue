<template>
	<button v-if="!isSlot && !isReadOnly"
		class="talk-button"
		:disabled="isCreateTalkRoomButtonDisabled"
		@click="createTalkRoom">
		Add video call link
	</button>
</template>

<script>
import { createTalkRoom, doesDescriptionContainTalkLink } from '../../../services/talkService.js'

export default {
	name: 'PropertyTalkButton',
	directives: {
		focus,
	},
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
		value: {
			type: String,
			default: '',
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
	},
	methods: {
		async createTalkRoom() {
			const NEW_LINE = '\r\n'
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
.talk-button{
	margin-top: 0.3rem !important;
	background-color: white !important;
	border-radius: 6px !important;
	border-color: var(--color-main-text) !important;
}
</style>
