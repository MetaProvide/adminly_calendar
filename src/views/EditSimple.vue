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
							<Download :size="20" decorative :class="'adminly-icon'" />
						</template>
						{{ $t('calendar', 'Export') }}
					</ActionLink>
					<ActionButton v-if="canDelete && !canCreateRecurrenceException" @click="deleteAndLeave(false)">
						<template #icon>
							<Delete :size="20" decorative :class="'adminly-icon'" />
						</template>
						{{ $t('calendar', 'Delete') }}
					</ActionButton>
					<ActionButton v-if="canDelete && canCreateRecurrenceException" @click="deleteAndLeave(false)">
						<template #icon>
							<Delete :size="20" decorative :class="'adminly-icon'" />
						</template>
						{{ $t('calendar', 'Delete this occurrence') }}
					</ActionButton>
					<ActionButton v-if="canDelete && canCreateRecurrenceException" @click="deleteAndLeave(true)">
						<template #icon>
							<Delete :size="20" decorative :class="'adminly-icon'" />
						</template>
						{{ $t('calendar', 'Delete this and all future') }}
					</ActionButton>
				</Actions>
				<button class="close-btn" @click="cancel"></button>
			</div>

			<PropertyCalendarPicker v-if="showCalendarPicker"
				:calendars="calendars"
				:calendar="selectedCalendar"
				:is-read-only="isReadOnly"
				:class="{ 'hidden' : !isNew }"
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
				:is-slot="isSlot"
				:is-read-only="isReadOnly"
				:can-modify-all-day="canModifyAllDay"
				:user-timezone="currentUserTimezone"
				@update-start-date="updateStartDate"
				@update-start-timezone="updateStartTimezone"
				@update-end-date="updateEndDate"
				@update-end-timezone="updateEndTimezone"
				@toggle-all-day="toggleAllDay" />

			<PropertyClientPicker :is-read-only="isReadOnly"
				:calendar-object-instance="calendarObjectInstance"
				:is-slot="isSlot" />

			<PropertyTalkButton :calendar-object-instance="calendarObjectInstance"
				:is-read-only="isReadOnly"
				:is-slot="isSlot" />

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
import PropertyTalkButton from '../components/Editor/Properties/PropertyTalkButton.vue'
import PropertyClientPicker from '../components/Editor/Properties/PropertyClientPicker.vue'
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
		PropertyTalkButton,
		PropertyClientPicker,
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
	async mounted() {
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
	padding: 2.75rem 2.5rem 2rem 2.5rem;
	border-radius: var(--adminly-border-radius);
	box-sizing: border-box;

	.property-select {
		margin-bottom: 1rem;
	}

	.property-select .property-select__input {
		label {
			font-size: 1rem;
			font-weight: 500;
			margin-left: 1rem;
			display: flex;
			align-items: center;
		}

		label::before {
			content: "";
			display: inline-block;
			width: 0.8rem;
			height: 0.8rem;
			border: 3px solid #fff;
			outline: 3px solid var(--color-border-dark);
			border-radius: 50%;
			margin-right: 1rem;
		}

		input[name='calendar-picker']:checked + label::before {
			background: #CB6BE6;
		}

		input[name='calendar-picker'] {
			position: absolute;
			opacity: 0;
		}
	}

	.property-title {
		margin-bottom: unset;
	}

	.property-title .property-title__input input {
		font-size: 0.875rem;
	}

	.property-title-time-picker__time-pickers {
		flex-direction: column;
	}

	.property-title-time-picker__time-pickers .mx-datepicker {
		width: 100%;
	}

	.adminly-buttons {
		display: flex;
		margin-top: 1rem;
		justify-content: center;

		button {
			padding: 0;
			width: 90px;
		}
		.primary + .update-all {
			margin-left: 5px;
		}
	}

	.adminly-buttons .event-popover__buttons {
		margin-top: 0;
	}

	.save-button button{
		border-radius: var(--adminly-border-radius);
		border: none;
		font-family: "Roc Grotesk", var(--font-face);
	}

	.cancel-button{
		background-color: unset;
		color: var(--color-primary-element);
		border: none;
		font-family: "Roc Grotesk", var(--font-face);
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

	input,
	.multiselect__tags,
	.mx-input,
	textarea {
		border: 1px solid var(--color-border-dark);
		border-radius: var(--border-radius);
	}

	.client-select .multiselect__tags,
	textarea:not(:disabled),
	div[contenteditable="true"]:not(:disabled) {
		&:active,
		&:focus,
		&:hover {
			border-color: var(--color-primary-element) !important;
		}
	}

	textarea {
		margin: 3px 0;
	}

	.client-select .input{
		border: none !important;
	}

	::placeholder, .multiselect__placeholder{
		color: var(--color-main-text);
		opacity: 0.6;
	}

	.close-btn{
		background-color: unset;
		border: none;
		background-image: url("../../img/close.svg");
		background-position: center;
		background-repeat: no-repeat;
		margin: 0;
		padding: 0;
		width: 44px;
		height: 44px;
		opacity: 0.7;

		&:hover {
			opacity: 1;
		}
	}

	.action-item__menutoggle:hover{
		background: white !important;
	}

	.property-repeat {
		margin-bottom: 1rem;
	}

	.repeat-option-set__label {
		margin-right: 0.5rem;
	}

	.repeat-option-set--end {
		margin-top: 0.25rem;
	}

	.property-alarm-list .bell-icon {
		background-image: url("../../img/bell.svg");
		background-repeat: no-repeat;
		background-position: center;
		background-size: 20px;

		svg {
			display: none;
		}
	}

	.property-repeat .pencil-icon {
		background-image: url("../../img/edit.svg");
		background-repeat: no-repeat;
		background-position: center;
		background-size: 20px;

		svg {
			display: none;
		}
	}

	.event-popover__top-right-actions {
		display: flex;
		padding-right: 1rem;
		opacity: 1 !important;
	}

	.hidden{
		display: none;
	}
}

.client-list-item{
	span{
		font-weight: 500;
	}
}

.multiselect__tags{
	border-radius: var(--border-radius);
	border-color: var(--color-border-dark) !important;
}

.multiselect--single, .talk-button{
	width: 100% !important;
}

.popover .action {
	height: 2rem;
    display: flex;
    align-items: center;

	.action-link,
	.action-button {
		font-weight: 500;
		opacity: 1;
		line-height: 2rem;
	}

	.action-button__longtext {
		align-self: center;
		padding: 0;
	}

	.adminly-icon {
		height: 2rem;
		width: 2rem;
		padding-left: 0.75rem;
	}

	&.active {
		.action-link,
		.action-button {
			color: white;
		}

		.adminly-icon,
		.material-design-icon.delete-icon {
			filter: brightness(0) invert(1);
		}
	}
}

.popover .popover__inner{
	border-bottom-left-radius: var(--adminly-border-radius-button);
	border-bottom-right-radius: var(--adminly-border-radius-button);
    padding-block: 0.5rem;
}

.adminly-icon,
.material-design-icon.delete-icon{
	background-repeat: no-repeat;
	background-position: center;
	background-size: 15px;
	svg{
		display: none;
	}
}

.delete-icon{
	background-image: url("../../img/delete.svg");
}

.download-icon{
	background-image: url("../../img/export.svg");
}
</style>
