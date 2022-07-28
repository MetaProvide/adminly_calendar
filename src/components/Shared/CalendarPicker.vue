<template>
	<div class="wrapper">
		<div v-for="calendar in calendars" class="option" :key="calendar.uid">
			<input
				v-if="value === calendar"
				type="radio"
				name="calendar-picker"
				:value="calendar"
				checked
				v-model="selected"
				:id="calendar.displayName"
			/>
			<input
				v-else
				type="radio"
				name="calendar-picker"
				:value="calendar"
				v-model="selected"
				:id="calendar.displayName"
			/>
			<label :for="calendar.displayName">{{ calendar.displayName }}</label>
		</div>
	</div>
</template>
<script>
export default {
	name: "CalendarPicker",
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
	display: flex;
	justify-content: space-around;
}

.option {
	display: flex;
	align-items: center;
}

input[type="radio"] {
	cursor: pointer;
	width: auto !important;
}
</style>
