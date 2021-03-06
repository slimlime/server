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
	<div id="app">
		<app-navigation :menu="menu" />
		<div id="app-content" class="app-settings-content" :class="{ 'with-app-sidebar': currentApp, 'icon-loading': loadingList }">
			<app-list :category="category" :app="currentApp" :search="searchQuery"></app-list>
			<div id="app-sidebar" v-if="id && currentApp">
				<app-details :category="category" :app="currentApp"></app-details>
			</div>
		</div>
	</div>
</template>


<script>
import appNavigation from '../components/appNavigation';
import appList from '../components/appList';
import Vue from 'vue';
import VueLocalStorage from 'vue-localstorage'
import Multiselect from 'vue-multiselect';
import api from '../store/api';
import AppDetails from '../components/appDetails';

Vue.use(VueLocalStorage)
Vue.use(VueLocalStorage)

export default {
	name: 'Apps',
	props: {
		category: {
			type: String,
			default: 'installed',
		},
		id: {
			type: String,
			default: '',
		}
	},
	components: {
		AppDetails,
		appNavigation,
		appList,
	},
	methods: {
		setSearch(query) {
			this.searchQuery = query;
		},
		resetSearch() {
			this.setSearch('');
		}
	},
	beforeMount() {
		this.$store.dispatch('getCategories');
		this.$store.dispatch('getAllApps');
		this.$store.dispatch('getGroups', {offset: 0, limit: -1});
		this.$store.commit('setUpdateCount', this.$store.getters.getServerData.updateCount)
	},
	mounted() {
		/** 
		 * Register search
		 */
		this.appSearch = new OCA.Search(this.setSearch, this.resetSearch);
	},
	data() {
		return {
			searchQuery: ''
		}
	},
	watch: {
		category: function (val, old) {
			this.setSearch('');
		}
	},
	computed: {
		loading() {
			return this.$store.getters.loading('categories');
		},
		loadingList() {
			return this.$store.getters.loading('list');
		},
		currentApp() {
			return this.apps.find(app => app.id === this.id );
		},
		categories() {
			return this.$store.getters.getCategories;
		},
		apps() {
			return this.$store.getters.getAllApps;
		},
		updateCount() {
			return this.$store.getters.getUpdateCount;
		},
		settings() {
			return this.$store.getters.getServerData;
		},

		// BUILD APP NAVIGATION MENU OBJECT
		menu() {
			// Data provided php side
			let categories = this.$store.getters.getCategories;
			categories = Array.isArray(categories) ? categories : [];

			// Map groups
			categories = categories.map(category => {
				let item = {};
				item.id = 'app-category-' + category.ident;
				item.icon = 'icon-category-' + category.ident;
				item.classes = [];							// empty classes, active will be set later
				item.router = {								// router link to
					name: 'apps-category',
					params: {category: category.ident}
				};
				item.text = category.displayName;

				return item;
			});


			// Add everyone group
			let defaultCategories = [
				{
					id: 'app-category-your-apps',
					classes: [],
					router: {name: 'apps'},
					icon: 'icon-category-installed',
					text: t('settings', 'Your apps'),
				},
				{
					id: 'app-category-enabled',
					classes: [],
					icon: 'icon-category-enabled',
					router: {name: 'apps-category', params: {category: 'enabled'}},
					text: t('settings', 'Active apps'),
				}, {
					id: 'app-category-disabled',
					classes: [],
					icon: 'icon-category-disabled',
					router: {name: 'apps-category', params: {category: 'disabled'}},
					text: t('settings', 'Disabled apps'),
				}
			];

			if (!this.settings.appstoreEnabled) {
				return {
					id: 'appscategories',
					items: defaultCategories,
				}
			}

			if (this.$store.getters.getUpdateCount > 0) {
				defaultCategories.push({
					id: 'app-category-updates',
					classes: [],
					icon: 'icon-download',
					router: {name: 'apps-category', params: {category: 'updates'}},
					text: t('settings', 'Updates'),
					utils: {counter: this.$store.getters.getUpdateCount}
				});
			}

			defaultCategories.push({
				id: 'app-category-app-bundles',
				classes: [],
				icon: 'icon-category-app-bundles',
				router: {name: 'apps-category', params: {category: 'app-bundles'}},
				text: t('settings', 'App bundles'),
			});

			categories = defaultCategories.concat(categories);

			// Set current group as active
			let activeGroup = categories.findIndex(group => group.id === 'app-category-' + this.category);
			if (activeGroup >= 0) {
				categories[activeGroup].classes.push('active');
			} else {
				categories[0].classes.push('active');
			}

			categories.push({
				id: 'app-developer-docs',
				classes: [],
				href: this.settings.developerDocumentation,
				text: t('settings', 'Developer documentation') + ' ↗',
			});

			// Return
			return {
				id: 'appscategories',
				items: categories,
				loading: this.loading
			}
		},
	}
}
</script>
