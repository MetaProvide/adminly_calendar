<!--
  - @copyright Copyright (c) 2019 Georg Ehrke <oc.list@georgehrke.com>
  -
  - @author Georg Ehrke <oc.list@georgehrke.com>
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
	<div class="calendar-picker-option">
		<label class="calendar-picker-option__label">
			<input type="radio" name="calendar-picker" />
			<span>{{ displayName }}</span>
		</label>
	</div>
</template>

<script>
export default {
	name: 'CalendarPickerOption',
	components: {
	},
	props: {
		color: {
			type: String,
			required: true,
		},
		displayName: {
			type: String,
			required: true,
		},
		owner: {
			type: String,
			required: true,
		},
		isSharedWithMe: {
			type: Boolean,
			required: true,
		},
	},
	computed: {
		/**
		 * Get the principal object of the calendar's owner
		 *
		 * @return {null | object}
		 */
		principal() {
			return this.$store.getters.getPrincipalByUrl(this.owner)
		},
		/**
		 * Gets the user-id of the calendar's owner
		 *
		 * @return {null | string}
		 */
		userId() {
			if (this.principal) {
				return this.principal.userId
			}

			return null
		},
		/**
		 * Gets the displayname of the calendar's owner
		 *
		 * @return {null | string}
		 */
		userDisplayName() {
			if (this.principal) {
				return this.principal.displayname
			}

			return null
		},
	},
}
</script>

<style lang="scss" scoped>
.calendar-picker-option__label {
	display: flex;
}

.calendar-picker-option__label span {
	margin: auto;
}
</style>
