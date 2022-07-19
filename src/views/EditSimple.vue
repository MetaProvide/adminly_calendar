<!--
  - @copyright Copyright (c) 2019 Georg Ehrke <oc.list@georgehrke.com>
  - @author Georg Ehrke <oc.list@georgehrke.com>
  - @author Richard Steinmetz <richard@steinmetz.cloud>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<Popover ref="popover"
		:open="isVisible"
		:auto-hide="false"
		:placement="placement"
		:boundaries-element="boundaryElement"
		open-class="adminly event-popover"
		trigger="manual">
		<template v-if="isLoading">
			<PopoverLoadingIndicator />
		</template>

		<template v-else-if="isError">
			<div class="event-popover__top-right-actions">
				<Actions>
					<ActionButton @click="cancel">
						<template #icon>
							<Close :size="20" decorative />
						</template>
						{{ $t('calendar', 'Close') }}
					</ActionButton>
				</Actions>
			</div>

			<EmptyContent>
				{{ $t('calendar', 'Event does not exist') }}
				<template #icon>
					<CalendarBlank :size="20" decorative />
				</template>
				<template #desc>
					{{ error }}
				</template>
			</EmptyContent>
		</template>

		<template v-else>
			<div class="event-popover__top-right-actions">
				<Actions v-if="isReadOnly">
					<ActionButton @click="showMore">
						<template #icon>
							<ArrowExpand :size="20" decorative />
						</template>
						{{ $t('calendar', 'Show more details') }}
					</ActionButton>
				</Actions>
				<Actions v-if="!isLoading && !isError && !isNew" :force-menu="true">
					<ActionLink v-if="!hideEventExport && hasDownloadURL"
						:href="downloadURL">
						<template #icon>
							<Download :size="20" decorative />
						</template>
						{{ $t('calendar', 'Export') }}
					</ActionLink>
					<ActionButton v-if="canDelete && !canCreateRecurrenceException" @click="deleteAndLeave(false)">
						<template #icon>
							<Delete :size="20" decorative />
						</template>
						{{ $t('calendar', 'Delete') }}
					</ActionButton>
					<ActionButton v-if="canDelete && canCreateRecurrenceException" @click="deleteAndLeave(false)">
						<template #icon>
							<Delete :size="20" decorative />
						</template>
						{{ $t('calendar', 'Delete this occurrence') }}
					</ActionButton>
					<ActionButton v-if="canDelete && canCreateRecurrenceException" @click="deleteAndLeave(true)">
						<template #icon>
							<Delete :size="20" decorative />
						</template>
						{{ $t('calendar', 'Delete this and all future') }}
					</ActionButton>
				</Actions>
			</div>

			<PropertyCalendarPicker v-if="showCalendarPicker"
				:calendars="calendars"
				:calendar="selectedCalendar"
				:is-read-only="isReadOnly"
				@select-calendar="changeCalendar"
				@switch-calendar="isSlotCheck"
				@current-calendar="isSlotCheck" />

			<PropertyTitle v-if="!isSlot"
				:value="title"
				:is-read-only="isReadOnly"
				@update:value="updateTitle" />

			<PropertyTitleTimePicker :start-date="startDate"
				:start-timezone="startTimezone"
				:end-date="endDate"
				:end-timezone="endTimezone"
				:is-all-day="isAllDay"
				:is-read-only="isReadOnly"
				:can-modify-all-day="canModifyAllDay"
				:user-timezone="currentUserTimezone"
				@update-start-date="updateStartDate"
				@update-start-timezone="updateStartTimezone"
				@update-end-date="updateEndDate"
				@update-end-timezone="updateEndTimezone"
				@toggle-all-day="toggleAllDay" />

			<PropertyText v-if="!isSlot"
				:is-read-only="isReadOnly"
				:prop-model="rfcProps.description"
				:value="description"
				@update:value="updateDescription" />

			<Repeat :calendar-object-instance="calendarObjectInstance"
				:recurrence-rule="calendarObjectInstance.recurrenceRule"
				:is-read-only="isReadOnly"
				:is-editing-master-item="isEditingMasterItem"
				:is-recurrence-exception="isRecurrenceException"
				@force-this-and-all-future="forceModifyingFuture" />

			<AlarmList v-if="!isSlot"
				:calendar-object-instance="calendarObjectInstance"
				:is-read-only="isReadOnly" />

			<div class="adminly-buttons">
				<Button class="cancel-button" @click="cancel">
					<template #icon>
						<Close :size="20" decorative />
					</template>
					{{ $t('calendar', 'Cancel') }}
				</Button>

				<SaveButtons v-if="!isReadOnly"
					class="event-popover__buttons save-button"
					:can-create-recurrence-exception="canCreateRecurrenceException"
					:is-new="isNew"
					:force-this-and-all-future="forceThisAndAllFuture"
					@save-this-only="saveAndLeave(false)"
					@save-this-and-all-future="saveAndLeave(true)" />
			</div>
		</template>
	</Popover>
</template>
<script>
import Actions from '@nextcloud/vue/dist/Components/Actions'
import ActionButton from '@nextcloud/vue/dist/Components/ActionButton'
import ActionLink from '@nextcloud/vue/dist/Components/ActionLink'
import EmptyContent from '@nextcloud/vue/dist/Components/EmptyContent'
import Popover from '@nextcloud/vue/dist/Components/Popover'
import EditorMixin from '../mixins/EditorMixin'
import PropertyTitle from '../components/Editor/Properties/PropertyTitle.vue'
import PropertyTitleTimePicker from '../components/Editor/Properties/PropertyTitleTimePicker.vue'
import PropertyCalendarPicker from '../components/Editor/Properties/PropertyCalendarPicker.vue'
import PropertyText from '../components/Editor/Properties/PropertyText.vue'
import SaveButtons from '../components/Editor/SaveButtons.vue'
import PopoverLoadingIndicator from '../components/Popover/PopoverLoadingIndicator.vue'
import { getPrefixedRoute } from '../utils/router.js'

import CalendarBlank from 'vue-material-design-icons/CalendarBlank.vue'
import Close from 'vue-material-design-icons/Close.vue'
import Delete from 'vue-material-design-icons/Delete.vue'
import Download from 'vue-material-design-icons/Download.vue'
import { mapState } from 'vuex'

import Repeat from '../components/Editor/Repeat/Repeat.vue'
import AlarmList from '../components/Editor/Alarm/AlarmList'

export default {
	name: 'EditSimple',
	components: {
		PopoverLoadingIndicator,
		SaveButtons,
		PropertyText,
		PropertyCalendarPicker,
		PropertyTitleTimePicker,
		PropertyTitle,
		Popover,
		Actions,
		ActionButton,
		ActionLink,
		EmptyContent,
		CalendarBlank,
		Close,
		Download,
		Delete,
		Repeat,
		AlarmList,
	},
	mixins: [
		EditorMixin,
	],
	computed: {
	  ...mapState({
		  hideEventExport: (state) => state.settings.hideEventExport,
	  }),
	},
	data() {
		return {
			placement: 'auto',
			hasLocation: false,
			hasDescription: false,
			boundaryElement: document.querySelector('#app-content > .fc'),
			isVisible: true,
			isSlot: false,
		}
	},
	watch: {
		$route(to, from) {
			// Update the popover position by updating its reference element.
			const isNew = to.name === 'NewPopoverView'
			const popover = this.$refs.popover.$children[0]
			popover.$_updatePopper(() => {
				popover.popperInstance.reference = this.getDomElementForPopover(isNew, to)
			})

			// Hide popover when changing the view until the user selects a slot again
			this.isVisible = to.params.view === from.params.view
		},
		calendarObjectInstance() {
			this.hasLocation = false
			this.hasDescription = false

			if (typeof this.calendarObjectInstance.location === 'string' && this.calendarObjectInstance.location.trim() !== '') {
				this.hasLocation = true
			}
			if (typeof this.calendarObjectInstance.description === 'string' && this.calendarObjectInstance.description.trim() !== '') {
				this.hasDescription = true
			}
		},
	},
	mounted() {
		window.addEventListener('keydown', this.keyboardCloseEditor)
		window.addEventListener('keydown', this.keyboardSaveEvent)
		window.addEventListener('keydown', this.keyboardDeleteEvent)
		this.$nextTick(() => {
			const isNew = this.$route.name === 'NewPopoverView'

			// V3 of V-Tooltip will have a prop to define the reference element for popper.js
			// For now we have to stick to this ugly hack
			// https://github.com/Akryum/v-tooltip/issues/60
			this.$refs.popover
				.$children[0]
				.$refs.trigger = this.getDomElementForPopover(isNew, this.$route)
		})
	},
	beforeDestroy() {
		window.removeEventListener('keydown', this.keyboardCloseEditor)
		window.removeEventListener('keydown', this.keyboardSaveEvent)
		window.removeEventListener('keydown', this.keyboardDeleteEvent)
	},
	methods: {
		showMore() {
			// Do not save yet
			this.requiresActionOnRouteLeave = false

			const params = Object.assign({}, this.$route.params)
			if (this.$route.name === 'NewPopoverView') {
				this.$router.push({ name: 'NewSidebarView', params })
			} else {
				this.$router.push({
					name: getPrefixedRoute(this.$route.name, 'EditSidebarView'),
					params,
				})
			}
		},
		getDomElementForPopover(isNew, route) {
			let matchingDomObject

			if (isNew) {
				matchingDomObject = document.querySelector('.fc-highlight')
				this.placement = 'auto'

				if (!matchingDomObject) {
					matchingDomObject = document.querySelector('.fc-event[data-is-new="yes"]')
				}
			} else {
				const objectId = route.params.object
				const recurrenceId = route.params.recurrenceId

				matchingDomObject = document.querySelector(`.fc-event[data-object-id="${objectId}"][data-recurrence-id="${recurrenceId}"]`)
				this.placement = 'auto'
			}

			if (!matchingDomObject) {
				matchingDomObject = document.querySelector('#app-navigation')
				this.placement = 'right'
			}

			if (!matchingDomObject) {
				matchingDomObject = document.querySelector('body')
				this.placement = 'auto'
			}

			return matchingDomObject
		},
		isSlotCheck(value) {
			this.isSlot = value.url.includes('appointment-slots')
		},
	},
}
</script>

<style lang="scss">
.adminly .calendar-picker-option {
	width: 100%;
}

.adminly.event-popover .popover__inner {
	max-width: 350px;
	width: 350px;
	padding: 2rem 1.5rem 1.5rem;
	border-radius: 1rem;

	.property-title-time-picker__time-pickers {
		flex-direction: column;
	}

	.property-title-time-picker__time-pickers .mx-datepicker {
		width: 100%;
	}

	.adminly-buttons {
		display: flex;
		margin-top: 1rem;
		justify-content: end;
	}

	.adminly-buttons .event-popover__buttons {
		margin-top: 0;
	}

	.save-button .primary{
		background-color: var(--adminly-light-blue);
	}

	.save-button button{
		border-radius: 8px;
		border: none;
	}

	.cancel-button{
		background-color: white;
		color: var(--adminly-light-blue);
		border: none;
	}

	> div > div:nth-child(2) .multiselect {
		input[type="radio"] {
			width: auto !important;
			cursor: pointer;
		}

		.multiselect__select,
		.multiselect__tags {
			display: none;
		}

		.multiselect__content-wrapper {
			display: inline-block !important;
			border: unset !important;
			position: unset;
		}

		.multiselect__content-wrapper .multiselect__content {
			display: flex !important;
			justify-content: space-around;
		}

		.multiselect__content-wrapper li > span,
		.multiselect__content-wrapper li > span.multiselect__option--highlight,
		.multiselect__content-wrapper li > span.multiselect__option--selected {
			background-color: unset !important;
			color: var(--color-main-text) !important;
			padding: 0;
		}
	}
}
</style>
