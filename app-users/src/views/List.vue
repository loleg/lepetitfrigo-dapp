<template>
	<div>

		<div v-show="shoppingCart.length" class="alert alert-success mt-3" role="alert">
				<strong>Shopping cart</strong>
				<hr>
				<ul>
					<li v-for="p in shoppingCart">
						<b style="float:right">{{ p.price }}</b>
						<span>{{ p.item }}</span>
					</li>
				</ul>
		</div>

		<div class="row product-list" v-show="!isLoading">
		  <div class="col-sm-4" v-for="product in products">
			  <div class="card-header text-center">
			    {{ product['Catégorie'] }}
			  </div>
		    <div class="card">
		      <div class="card-body" v-bind:style="product.src">
						<div class="card-content">
			        <p class="card-text">
								{{ product['Types'] }}
							</p>
							<h5 class="card-price" v-show="product['Prix /kg']">{{ product['Prix /kg'] }} /kg</h5>
							<h5 class="card-price">{{ product['Prix Unité '] }}</h5>
						</div>
		        <center>
							<a class="btn btn-warning" v-on:click="performBuy(product)">Je l'prends!</a>
						</center>
		      </div>
		    </div>
		  </div>
		</div>

		<div class="clearfix"></div><hr>

		<button class="btn btn-primary float-right mt-2" @click="reloadList">Refresh</button>
    <h1 class="title">Communauté</h1>

    <h2 v-show="!bcConnected">Connecting, please wait.</h2>

    <h2 v-show="(isLoading && bcConnected)">Loading...</h2>

    <table class="table table-striped" v-show="!isLoading">
        <thead class="thead-dark">
            <tr>
                <th>User</th>
								<th>Status</th>
								<th>Balance</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="user in users">
                <td>
									<b>{{ user[1] }}</b><br>
									<small># {{ user[0].toNumber() }}</small>
								</td>
                <td>
									{{ toAscii(user[3]) }}<br>
									<small>{{ toDate( user[6].toNumber() ) }}</small>
								</td>
								<td>{{ user[2] }}</td>
            </tr>
        </tbody>
    </table>

	</div>
</template>

<script>
    // importing common function
    import mixin from '../libs/mixinViews';
		import Papa from 'papaparse';

		// const urlProductsDB = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS8bZeOC0izNWaCWRPtt4GZV0VSF-g7N1Pf2ZOL-pcC76IL24nhugnlwfjtzONIYhxSlWpC92F6Dpqn/pub?gid=1661288941&single=true&output=csv";
		const urlProductsDB = "/static/produits.csv";

    /**
     * List view component: this component shows list of the registered users
     * and their statuses.
     */
    export default {
        mixins: [mixin],

        data() {
            return {
                users: [], // array that stores all the registered users
								products: [], // all the products from the shop
                isLoading: true, // true when the user list is loading form the blockchain
                bcConnected: false, // blockchain is connected ()
                tmoConn: null, // contain the intervalID given by setInterval
								shoppingCart: []
            }
        },

        methods: {
            /**
             * Get the list of the registered users once the connection to the
             * blockchain is established.
             */
            getUserList() {
                if (this.blockchainIsConnected()) {
                    // it shows the loading message
                    this.isLoading = true;

                    // stopping the interval
                    clearInterval(this.tmoConn);

										this.isRegistered().then(res => {
												if (res) {
				                    // getting all the users from the blockchain
				                    this.getAllUsers(userProfile => {
				                        this.isLoading = false;
				                        this.users.push(userProfile);
				                    })
												} else {
														// redirecting to the profile page
														this.$router.push("register");
												}
										});

                }
            },

            /**
             * It reloads the user list.
             */
            reloadList() {
                this.users = [];
                this.getUserList();
            },

						/**
						 * Get all users.
						 */
						getAllUsers(callback) {
							// getting the total number of users stored in the blockchain
							// calling the method totalUsers from the smart contract
							window.bc.contract().totalUsers((err, total) => {
			                    var tot = 0;
								if (total) tot = total.toNumber();

								if (tot > 0) {
									// getting the user one by one
									for (var i=1; i<=tot; i++) {
										window.bc.contract().getUserById.call(i, (error, userProfile) => {
											callback(userProfile);
										});
									} // end for
								} // end if
							}); // end totalUsers call
						},

						/**
						 * Get product list.
						 */
						getProducts() {
							let self = this;
							Papa.parse(urlProductsDB, {
								download: true,
								header: true,
								skipEmptyLines: true,
								complete: function(results) {
									// console.log(results);
									results.data.forEach(function(img) {
										img.src = 'background-image:url("/static/produits/' + img.Image + '.jpg")';
									});
									self.products = results.data;
								}
							});
						},

						/**
             * Updates the user's details when the button is pressed.
             *
             * @return {void}
             */
            performBuy(product) {

							let p_Type = product['Types'];
							let p_Price = 0;

							if (product['Prix /kg']) {
								let n = prompt('Poids?');
								if (isNaN(parseFloat(n)) || !isFinite(n)) {
									return; // cancel
								}
								p_Price = parseFloat(n) * parseFloat(product['Prix /kg']);

							} else if (product['Prix Unité ']) {
								let n = prompt('Montant?');
								if (isNaN(parseInt(n)) || !isFinite(n)) {
									return; // cancel
								}
								p_Price = parseInt(n) * parseFloat(product['Prix Unité ']);
							}
							if (p_Price == 0) return;

							// console.log(p_Type, p_Price);

                this.submitting = true;
                this.errorMessage = null;
                this.successSave = false;
								let self = this;


                // calling the method updateUser from the smart contract
                window.bc.getMainAccount().then(account => {
									window.bc.contract().getOwnProfile.call({ from: account },
											(error, userDet) => {
													if (userDet) {
															let userName = userDet[1];
															let userTime = userDet[2];
															let userStatus = p_Type;
															userTime -= p_Price;

					                    window.bc.contract().updateUser(
					                        userName,
					                        userTime,
					                        userStatus,
					                        {
					                            from: account,
					                            gas: 800000
					                        },
					                        (err, txHash) => {
																		self.shoppingCart.push({
																			item: userStatus,
																			price: p_Price
																		})
																	}
					                    )

													} else {
														window.alert('Could not update!');
													}
										})
                })
                .catch(error => window.alert(error));
            }

        }, // end methods

        created() {
            // it tries to get the user list from the blockchian once
            // the connection is established
            this.tmoConn = setInterval(() => {
                this.getUserList();
								this.getProducts();
            }, 1000);
        }
    }
</script>
