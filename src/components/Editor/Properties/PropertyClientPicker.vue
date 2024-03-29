<template>
	<div class="client-select">
		<Multiselect v-if="writableEvent"
			v-model="selectedClient"
			label="name"
			track-by="email"
			placeholder="Select client"
			:options="clientSearchList"
			:searchable="true"
			:internal-search="false"
			:show-no-results="true"
			:show-no-options="false"
			@search-change="findClients"
			@select="addAttendee">
			<template #option="{ option }">
				<div class="client-list-item">
					<span>
						{{ option.name }}
					</span>
					<p>{{ option.email }}</p>
				</div>
			</template>
		</Multiselect>
		<button v-if="this.attendees.length" class="remove" @click="removeAttendee()"></button>
	</div>
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
			selectedClient: [],
			clientDataDescription: '',
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
			if (this.attendees.length) {
				this.removeAttendee()
			}

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

			this.clientDataDescription = selectedValue.phoneNumber ?
								NEW_LINE + selectedValue.name + NEW_LINE
								+ selectedValue.phoneNumber + NEW_LINE
								+ selectedValue.email + NEW_LINE :
								NEW_LINE + selectedValue.name + NEW_LINE
								+ selectedValue.email + NEW_LINE;

			this.$store.commit('changeDescription', {
				calendarObjectInstance: this.calendarObjectInstance,
				description: this.calendarObjectInstance.description ?
								this.calendarObjectInstance.description + this.clientDataDescription
								: this.clientDataDescription,
			})

			this.clientEmail = ''
		},
		removeAttendee() {
			const attendee = this.attendees.pop()
			const newDescription = this.calendarObjectInstance.description.replace(this.clientDataDescription, '')

			this.$store.commit('changeDescription', {
				calendarObjectInstance: this.calendarObjectInstance,
				description: newDescription,
			})

			this.$store.commit('removeAttendee', {
				calendarObjectInstance: this.calendarObjectInstance,
				attendee,
			})
			this.selectedClient = []
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
<style>
.client-select {
	display: flex;
	flex-direction: column;
	margin: 3px 0;
}

.client-select .remove {
	align-self: flex-end;
	background-color: unset;
	background-image: url("../../../../img/close.svg");
	background-position: center;
	background-repeat: no-repeat;
	border: unset;
	display: flex;
	margin: 4px 0 !important;
	padding: 6px 16px;
	position: absolute;
	z-index: 5;
}

.client-select .multiselect__single {
	color: var(--color-main-text) !important;
}
</style>
