<template>
	<div id="clients" class="clients">

		<create-client-form
			v-if="showCreateClientForm"
			@close="closeClientModalForm"
			@save-client="addClient"
			:waiting="requestProcessing"
			:edit="editClient"
		></create-client-form>

		<div class="row">
			<div class="col-md-9">
			    <h3 class="page-h3 no-mar-pad">
			    	clients
			    </h3>
			</div>
			<div class="col-md-3">
                <div class="action-button">
                    <a
                    	href="#"
                    	@click.prevent="showCreateClientForm = true"
                    	class="add-site full-width"
                    >
                        <i></i>
                        <span>Client</span>
                    </a>
                </div>
			</div>
		</div>

		<div class="clear mb-30"></div>

		<div class="actions-panel current-box-shadow">
			<div class="row">
				<div class="col-sm-6">
					<input
						v-model="quickSearch"
						class="white-bg"
						type="text"
						placeholder="Start typing a name, email or phone..."
					/>
				</div>
				<div v-show="quickSearch && quickSearch.length > 1" class="col-sm-3">
					<div class="actions-wrap">
						<a @click.prevent="quickSearch = ''" href="#" class="remove full-width">
							<i></i>
							Clean
		               	</a>
		            </div>
		        </div>
	        </div>
	    </div>

	    <div class="items-count pull-right">{{ filteredClients.length }} клиент(-ов)</div>

		<table v-show="clients.length" class="dark-header">
			<thead>
				<tr class="table-header">
					<td v-for="(col, index) in gridColumns">
						<a
							@click.prevent="sortBy(col.key)"
							class="dashed-link"
							href="#"
							:title="'Sort by ' + col.label"
						>
							{{ col.label }}
							<i
								v-show="col.key === sortKey"
								:class="sortOrders[col.key] === 1 ? 'up' : 'down'"
							></i>
						</a>
					</td>
					<td>Date of birth</td>
					<td class="text-center">Notes</td>
					<td class="text-center">Action</td>
				</tr>
			</thead>
			<tbody>
				<tr v-for="(client, index) in filteredClients">
					<td><span>Name:</span>{{ client.full_name }}</td>
					<td><span>Email:</span>{{ client.email || '-' }}</td>
					<td><span>Phone:</span>{{ client.phone || '-' }}</td>
					<td><span>Date of birth:</span>{{ client.birth_day || '-' }}</td>
					<td><span>Notes:</span>{{ client.notes || '-' }}</td>
					<td>
						<i @click="removeClient(client.id)" class="custom-icon remove"></i>
						<i @click.prevent="showEditForm(client)" class="custom-icon edit"></i>
					</td>
				</tr>
			</tbody>
		</table>

		<div v-show="!clients.length" class="items-count">You have not added any clients yet.</div>

	</div>
</template>

<script>
	var API_URLS = JSON.parse(document.getElementById("body").getAttribute("api-urls"));
	var CREATE_CLIENT_URL = API_URLS.booking.api.createClient,
		FETCH_CLIENTS_URL = API_URLS.booking.api.getClientList,
		REMOVE_CLIENT_URL = API_URLS.booking.api.removeClient;

	import {HTTP} from './main';

	export default {
		name: 'clients',
		created() {
			this.fetchClients();
		},
		data() {
			var gridColumns = [
				{
					key: 'full_name',
					label: 'Name'
				},
				{
					key: 'email',
					label: 'Email'
				},
				{
					key: 'phone',
					label: 'Phone'
				}
			];
			var sortOrders = {};
			gridColumns.forEach(function (item) {
      			sortOrders[item.key] = 1;
    		});

			return {
				showCreateClientForm: false,
				quickSearch: '',
			    sortKey: '',
			    sortOrders: sortOrders,
			    gridColumns: gridColumns,
				clients: [],
				requestProcessing: false,
				editClient: false
			}
		},
		methods: {
			/**
			* Gets the clients from the API endpoint.
			*
	        * @method
	        * @instance
			*/
			fetchClients() {
				var self = this;
				HTTP.get(FETCH_CLIENTS_URL)
					.then(function (response) {
						self.clients = response.data;
					})
					.catch(function (error) {
						console.log(error);
					});
			},
			/**
			* Close the add new client popup form.
			*
	        * @method
	        * @instance
			*/
			closeClientModalForm() {
				this.editClient = false;
				this.showCreateClientForm = false;
			},
			/**
			* Show edit client popup form.
			*
	        * @method
	        * @instance
	        * @param {Object} client - Gives client to edit.
			*/
			showEditForm(client) {
				this.editClient = client;
				this.showCreateClientForm = true;
			},
			/**
			* Save the new client according to given data.
			*
	        * @method
	        * @instance
	        * @param {Object} data - Data for a new client.
			*/
			addClient(data) {
				var self = this;
				var editID = self.editClient && self.editClient.id;

				// Start request
				self.requestProcessing = true;

				HTTP.post(CREATE_CLIENT_URL, data)
					.then(function (response) {

						// End request
						self.requestProcessing = false;

						// Close the form
						self.closeClientModalForm();

						if (editID) {
							self.clients = self.clients.filter(function (item) {
								return item.id !== editID;
							})
						}
						self.clients.push(response.data);
					})
					.catch(function (error) {
						console.log(error);
					});
			},
			/**
			* Remove a client from the list by given ID.
			*
	        * @method
	        * @instance
	        * @param {Number} id - The ID for current client.
			*/
			removeClient(id) {
				var self = this;

				HTTP.post(REMOVE_CLIENT_URL, {id: id})
					.then(function (response) {

						self.clients = self.clients.filter(function(client) {
                            return client.id !== id;
                        });
					})
					.catch(function (error) {
						console.log(error);
					});
			},
			/**
			* Applies field key to sort list of clients.
			*
	        * @method
	        * @instance
	        * @param {String} key - Field key.
			*/
			sortBy(key) {
				this.sortKey = key;
				this.sortOrders[key] = this.sortOrders[key] * -1;
			},
		},
		computed: {
			/**
			* Applies filters for list of clients.
			*
	        * @method
	        * @instance
			*/
			filteredClients() {
				var clients = this.clients;
				var searchKey = this.quickSearch && this.quickSearch.toLowerCase();

				if (searchKey && searchKey.length > 1) {
					clients = clients.filter(function(client) {
						return client.full_name.toLowerCase().indexOf(searchKey) > -1 || client.email.toLowerCase().indexOf(searchKey) > -1 || client.phone.toLowerCase().indexOf(searchKey) > -1;
					});
				}

				var sortKey = this.sortKey;
				var order = this.sortOrders[sortKey] || 1;

				if (sortKey) {
			        clients = clients.slice().sort(function (a, b) {
			          a = a[sortKey];
			          b = b[sortKey];
			          return (a === b ? 0 : a > b ? 1 : -1) * order;
			        })
				}

				return clients;
			}
		}
	}
</script>