<!--
  - @copyright Copyright (c) 2018 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div class="section" v-bind:class="{ selected: isSelected }" v-on:click="showAppDetails">
		<div class="app-image app-image-icon" v-on:click="showAppDetails">
			<div v-if="(listView && !app.preview) || (!listView && !app.screenshot)" class="icon-settings-dark"></div>

			<svg v-if="listView && app.preview" width="32" height="32" viewBox="0 0 32 32">
				<defs><filter :id="filterId"><feColorMatrix in="SourceGraphic" type="matrix" values="-1 0 0 0 1 0 -1 0 0 1 0 0 -1 0 1 0 0 0 1 0"></feColorMatrix></filter></defs>
				<image x="0" y="0" width="32" height="32" preserveAspectRatio="xMinYMin meet" :filter="filterUrl" :xlink:href="app.preview" class="app-icon"></image>
			</svg>

			<img v-if="!listView && app.screenshot" :src="app.screenshot"  width="100%" />
		</div>
		<div class="app-name" v-on:click="showAppDetails">
			{{ app.name }}
		</div>
		<div class="app-summary" v-if="!listView">{{ app.summary }}</div>
		<div class="app-version" v-if="listView">
			<span v-if="app.version">{{ app.version }}</span>
			<span v-else-if="app.appstoreData.releases[0].version">{{ app.appstoreData.releases[0].version }}</span>
		</div>

		<div class="app-level">
			<span class="official icon-checkmark" v-if="app.level === 200"
				  v-tooltip.auto="t('settings', 'Official apps are developed by and within the community. They offer central functionality and are ready for production use.')">
				{{ t('settings', 'Official') }}</span>
			<app-score v-if="!listView" :score="app.score"></app-score>
		</div>

		<div class="actions">
			<div class="warning" v-if="app.error">{{ app.error }}</div>
			<div class="icon icon-loading-small" v-if="loading(app.id)"></div>
			<input v-if="app.update" class="update" type="button" :value="t('settings', 'Update to {update}', {update:app.update})" v-on:click="update(app.id)" :disabled="installing || loading(app.id)" />
			<input v-if="app.canUnInstall" class="uninstall" type="button" :value="t('settings', 'Remove')" v-on:click="remove(app.id)" :disabled="installing || loading(app.id)" />
			<input v-if="app.active" class="enable" type="button" :value="t('settings','Disable')" v-on:click="disable(app.id)" :disabled="installing || loading(app.id)" />
			<input v-if="!app.active" class="enable" type="button" :value="enableButtonText" v-on:click="enable(app.id)" v-tooltip.auto="enableButtonTooltip" :disabled="!app.canInstall || installing || loading(app.id)" />
		</div>
	</div>
</template>

<script>
	import Multiselect from 'vue-multiselect';
	import AppScore from './appScore';
	import AppManagement from '../appManagement';
	import SvgFilterMixin from '../svgFilterMixin';

	export default {
		name: 'appItem',
		mixins: [AppManagement, SvgFilterMixin],
		props: {
			app: {},
			category: {},
			listView: {
				type: Boolean,
				default: true,
			}
		},
		watch: {
			'$route.params.id': function (id) {
				this.isSelected = (this.app.id === id);
			}
		},
		components: {
			Multiselect,
			AppScore,
		},
		data() {
			return {
				isSelected: false,
				scrolled: false,
			};
		},
		mounted() {
			this.isSelected = (this.app.id === this.$route.params.id);
		},
		computed: {

		},
		watchers: {

		},
		methods: {
			showAppDetails(event) {
				if (event.currentTarget.tagName === 'INPUT' || event.currentTarget.tagName === 'A') {
					return;
				}
				this.$router.push({
					name: 'apps-details',
					params: {category: this.category, id: this.app.id}
				});
			},
			prefix(prefix, content) {
				return prefix + '_' + content;
			},
		}
	}
</script>
