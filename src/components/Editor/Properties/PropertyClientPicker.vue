<template>
	<Multiselect v-if="writableEvent"
		v-model="selectedValue"
		label="name"
		track-by="email"
		:options="clientSearchList"
		:searchable="true"
		:internal-search="false"
		:show-no-results="true"
		:show-no-options="false"
		@select="addAttendee"
		@search-change="findClients"
		placeholder='Select client'>
		<template #option="{ option }">
			<div class="client-list-item">
				<span>
					{{ option.name }}
				</span>
				<p>{{ option.email }}</p>
			</div>
		</template>
	</Multiselect>
</template>

<script>
import Multiselect from '@nextcloud/vue/dist/Components/Multiselect'
import { ClientsUtil, NEW_LINE } from '../../../utils.js'

export default {
	name: 'PropertyClientPicker',
	components: {
		Multiselect,
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
	},
	data() {
		return {
			clientsList: [],
			clientSearchList: [],
			selectedValue: [],
		}
	},
	computed: {
		attendees() {
			return this.calendarObjectInstance.attendees.filter(attendee => {
				return !['RESOURCE', 'ROOM'].includes(attendee.attendeeProperty.userType)
			})
		},
		writableEvent() {
			return !this.isSlot && !this.isReadOnly
		},
	},
	async mounted() {
		this.clientsList = await ClientsUtil.fetchClients()
		this.clientSearchList = this.clientsList
	},
	methods: {
		addAttendee(selectedValue) {
			if(this.attendees.length)
				this.removeAttendee(this.attendees.pop())

			this.$store.commit('addAttendee', {
				calendarObjectInstance: this.calendarObjectInstance,
				commonName: selectedValue.email,
				uri: selectedValue.email,
				calendarUserType: 'INDIVIDUAL',
				participationStatus: 'NEEDS-ACTION',
				role: 'REQ-PARTICIPANT',
				rsvp: true,
				language: null,
				timezoneId: null,
				organizer: this.$store.getters.getCurrentUserPrincipal,
			})

			let newDescription
			if (!this.calendarObjectInstance.description) {
				newDescription = NEW_LINE + selectedValue.name + NEW_LINE
								+ selectedValue.phoneNumber + NEW_LINE
								+ selectedValue.email
			} else {
				newDescription = this.calendarObjectInstance.description + NEW_LINE
								+ selectedValue.name + NEW_LINE
								+ selectedValue.phoneNumber + NEW_LINE
								+ selectedValue.email
			}

			this.$store.commit('changeDescription', {
				calendarObjectInstance: this.calendarObjectInstance,
				description: newDescription,
			})

			this.clientEmail = ''
		},
		removeAttendee(attendee) {
			const oldDescription = this.calendarObjectInstance.description
			const lines = oldDescription.split(NEW_LINE)
			const email = attendee.uri
			const emailIndex = lines.indexOf(email)

			lines.splice(emailIndex - 2, 3)

			const newDescription = lines.join(NEW_LINE)

			this.$store.commit('changeDescription', {
				calendarObjectInstance: this.calendarObjectInstance,
				description: newDescription,
			})

			this.$store.commit('removeAttendee', {
				calendarObjectInstance: this.calendarObjectInstance,
				attendee,
			})
		},
		findClients(query) {
			this.clientSearchList = this.clientsList.filter((p) => {
				return (
					p.name
						.toLowerCase()
						.indexOf(query.toLowerCase()) !== -1
				);
			});
		},
	},
}
</script>
