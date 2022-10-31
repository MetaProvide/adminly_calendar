<!--
  - @copyright Copyright (c) 2019 Georg Ehrke <oc.list@georgehrke.com>
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
	<div class="action-item">
		<div class="button-wrapper">
			<button v-for="view in views"
				:key="view.id"
				class="view-selector-button"
				:class="{ active: isActive(view.id) }"
				@click="selectView(view.id)">
				<span class="action-button__text">{{ view.label }}</span>
			</button>
		</div>
	</div>
</template>

<script>
export default {
	name: 'AppNavigationHeaderViewMenu',
	computed: {
		views() {
			return [{
				id: 'timeGridDay',
				icon: 'ViewDay',
				label: 'Day',
				// label: this.$t('calendar', 'Day'),
			}, {
				id: 'timeGridWeek',
				icon: 'ViewWeek',
				label: 'Week',
				// label: this.$t('calendar', 'Week'),
			}, {
				id: 'dayGridMonth',
				icon: 'ViewModule',
				label: 'Month',
				// label: this.$t('calendar', 'Month'),
			}]
		},
		shortKeyConf() {
			return {
				timeGridDay: ['d'],
				timeGridDay_Num: [1],
				timeGridWeek: ['w'],
				timeGridWeek_Num: [2],
				dayGridMonth: ['m'],
				dayGridMonth_Num: [3],
				listMonth: ['l'],
				listMonth_Num: [4],
			}
		},
	},
	methods: {
		selectView(viewName) {
			const name = this.$route.name
			const params = Object.assign({}, this.$route.params, {
				view: viewName,
			})

			// Don't push new route when view didn't change
			if (this.$route.params.view === viewName) {
				return
			}

			this.$router.push({ name, params })
		},
		selectViewFromShortcut(event) {
			this.selectView(event.srcKey.split('_')[0])
		},
		isActive(viewId) {
			return this.$route.path.includes(viewId)
		},
	},
}
</script>
