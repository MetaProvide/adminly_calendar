<template>
	<div class="wrapper">
		<div v-for="calendar in calendars" class="options" :key="calendar.uid">
			<label class="calendar-picker-option__label">
				<input
					v-if="value === calendar"
					type="radio"
					name="calendar-picker"
					:value="calendar"
					checked
					v-model="selected"
				/>
				<input
					v-else
					type="radio"
					name="calendar-picker"
					:value="calendar"
					v-model="selected"
				/>
				<span>{{ calendar.displayName }}</span>
			</label>
		</div>
	</div>
	<!-- <Multiselect label="displayName"
			track-by="url"
			:disabled="isDisabled"
			:options="calendars"
			:value="value"
			:multiple="multiple"
			@change="change"
			@remove="remove"
		>
			<template #singleLabel="{ option }">
				<CalendarPickerOption v-bind="option" />
			</template>
			<template #option="{ option }">
				<CalendarPickerOption v-bind="option" />
			</template>
			<template #tag="{ option }">
				<div class="calendar-picker__tag">
					<CalendarPickerOption v-bind="option" />
				</div>
			</template>
		</Multiselect> -->
</template>
<script>
import Multiselect from "@nextcloud/vue/dist/Components/Multiselect";
import CalendarPickerOption from "./CalendarPickerOption.vue";

export default {
	name: "CalendarPicker",
	components: {
		CalendarPickerOption,
		Multiselect,
	},
	props: {
		value: {
			type: [Object, Array],
			required: true,
		},
		calendars: {
			type: Array,
			required: true,
		},
		showCalendarOnSelect: {
			type: Boolean,
			default: false,
		},
		multiple: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			selected: this.value,
		};
	},
	computed: {
		isDisabled() {
			return this.calendars.length < 2;
		},
	},
	watch: {
		/**
		 * TODO: this should emit the calendar id instead
		 *
		 * @param {object} newCalendar The selected calendar
		 */
		selected(newCalendar) {
			this.$emit("switch-calendar", newCalendar);
			if (!newCalendar) {
				return;
			}

			if (this.showCalendarOnSelect && !newCalendar.enabled) {
				this.$store.dispatch("toggleCalendarEnabled", {
					calendar: newCalendar,
				});
			}

			this.$emit("select-calendar", newCalendar);
		},
	},
	methods: {

		remove(calendar) {
			if (this.multiple) {
				this.$emit("remove-calendar", calendar);
			}
		},
	},
};
</script>

<style lang="scss" scoped>
::v-deep .multiselect__tags {
	margin: 3px 0;
}

.calendar-picker__tag {
	border: 1px solid var(--color-border);
	border-radius: var(--border-radius);
	padding: 0 5px;
}

.calendar-picker__tag + .calendar-picker__tag {
	margin-left: 5px;
}

.wrapper {
	display: inline-block !important;
}

ul {
	display: flex;
	justify-content: space-around;
}

li {
	position: relative;
	display: flex;
	align-items: center;
	background-color: transparent;
}
</style>
