# Billing

There are a various way to make money out of app such as having app as paid, ads, in-app purchases (IAP). Among them IAP is most preferred option, because one can earn more money by selling virtual products than the app itself.

In this post, you can expect to learn Registering products with the Store, Configuring plugin, Implementing the Billing feature and finally we will windup with a note on Testing.

Getting Started
Before we proceed into registration, let me share info about billing products. In our plugin 2 types of billing products are supported. They are:

• Consumables: These item types gets exhausted with use. So user is allowed to buy it multiple times. For e.g.: Ammo, Health packs etc
• Non-Consumables: These item types can be bought only once, as user owns it permanently in all the devices. For e.g.: Bonus level, Weapons etc.

Register With Store
Store won’t recognise your products, unless it finds the records with it. So lets create records by registering and assigning an unique identifier for each product. I won’t run you through the registration process, instead I will share references which will guide you through it.

On iOS, registration of products is done through iTunes Connect. For setup information, please refer to Creating In-app Purchase Products. And coming to Android, all the product records are managed using Google Play Developer Console, for more information see Administering In-app Billing.

Configure Plugin
By now, you might be aware of NPSettings. Incase if its new to you, read this post. 
In your Unity project, open NPSettings and select Application Settings. Here you need to set Uses Billing checkbox as enabled. This step will activate the billing feature and make it ready for use. Next, select Billing Settings. On doing so, it will display all the config options available for Billing feature. Let us take a brief look into it:



1. Products: If your app keeps purchasable item information within itself, you can use this property to save those information. Later on, you can call NPSettings.Billing.Products to access this value from your code.
2. iOS Settings: Here, first thing you will notice is “Supports Receipts Validation”. This optional feature allows you to verify receipts of completed transactions. The receipt is a record of purchase made from within the application. Enabling receipt validation, adds one more level of security to avoid unauthorised purchases. By default, our plugin contacts Apple Server to verify transaction receipts. However you are free to override this behaviour, by providing URL of the server to which receipt data has to be sent for verification. For more information, see Validating Receipts With App Store.
3. Android Settings: You need to provide public key provided by Google Play services. This key is used for verifying signatures. For more information, read Securing Your Application.

Thats it! Save all your NPSettings changes by clicking Save button and also save the project.

Implementing Feature
Now, you are ready to begin implementation of Billing feature. Our cross-platform Billing component acts as a bridge between your application and Store. Use NPBinding.Billing to access billing features from your code.

Note: All the classes, datatypes related to the plugin are placed under namespace VoxelBusters.NativePlugins. Add this namespace in your script before accessing any of the features.
The interaction between user, your app and the Store is simple and straightforward. Take a look at the following illustration to get a better understanding about the workflow:



Displaying Products
First thing, we need to do is dynamically load information of all the registered virtual products from the Store.

public void RequestBillingProducts ()
{
	NPBinding.Billing.RequestForBillingProducts(NPSettings.Billing.Products);

	// At this point you can display an activity indicator to inform user that task is in progress
}

private void OnEnable ()
{
	// Register for callbacks
	Billing.DidFinishProductsRequestEvent		+= OnDidFinishProductsRequest;
}

private void OnDisable ()
{
	// Deregister for callbacks
	Billing.DidFinishProductsRequestEvent		-= OnDidFinishProductsRequest;
}

private void OnDidFinishProductsRequest (BillingProduct[] _regProductsList, string _error)
{
	// Hide activity indicator

	// Handle response
	if (_error != null)
	{		
		// Something went wrong
	}
	else 
	{	
		// Inject code to display received products
	}
}
And to do that, we call RequestForBillingProducts. The Store verifies all the requested product identifiers and returns localised information of all the requested products by calling DidFinishProductsRequestEvent. Event data might also hold the description of the error that occurred while processing this request (if any).

Make Purchase
At this point, we have an user interface to display all the purchasable products. Now lets add the code to make purchase. 

Consider, your user interface knows the product that user wants to purchase. What next? That is what we will implement now. In your BillingManager.cs add this code:

public void BuyItem (BillingProduct _product)
{
	if (NPBinding.Billing.IsProductPurchased(_product.ProductIdentifier))
	{
		// Show alert message that item is already purchased

		return;
	}

	// Call method to make purchase
	NPBinding.Billing.RequestForBillingProducts(NPSettings.Billing.Products);

	// At this point you can display an activity indicator to inform user that task is in progress
}
In BuyItem method, first we need to determine if specified product was previously purchased. As mentioned earlier, IAP has Non-Consumable products, which can be bought only once and it is our responsibility to ensure that it is available to a user in all the devices. Trying to purchase same item multiple times violates Store guidelines. So it is recommended to call IsProductPurchased before making a purchase request. 

After determining that product is not yet purchased, we can make a payment request to the Store by calling BuyProduct. The Store shows a prompt asking for confirmation and then requests to enter account details. And after getting required information, Store processes the purchase request. Boom! Money in your bank! 

But we are not done yet! You need to give back the promised content. And for that, you need to parse through the transaction details received from DidReceiveTransactionInfoEvent and identify finished transaction. 

private void OnDidFinishTransaction (BillingTransaction[] _transactions, string _error)
{
	if (_transactionList != null)
	{
		foreach (BillingTransaction _eachTransaction in _transactionList)
		{
			if (_eachTransaction.VerificationState == eBillingTransactionVerificationState.SUCCESS)
			{
				if (_eachTransaction.TransactionState == eBillingTransactionState.PURCHASED)
				{
					// Your code to handle purchased products
				}
				else if (_eachTransaction.TransactionState == eBillingTransactionState.RESTORED)
				{
					// Your code to handle restored products
				}
			}
		}
	}
}
Also, update your OnEnable, OnDisable method with transaction event registration code.

private void OnEnable ()
{
	// Register for callbacks
	Billing.DidFinishProductsRequestEvent	+= OnDidFinishProductsRequest;
	Billing.DidReceiveTransactionInfoEvent	-= OnDidFinishTransaction;
}

private void OnDisable ()
{
	// Deregister for callbacks
	Billing.DidFinishProductsRequestEvent	-= OnDidFinishProductsRequest;
	Billing.DidReceiveTransactionInfoEvent	-= OnDidFinishTransaction;
}
Restore Purchases
In this section we will talk about making Non-Consumable purchases available to the user in all the devices. 

If your app has Non-Consumable products, you must implement a mechanism to restore old purchases. In fact, your app might get rejected if you fail to do so.

In previous step, we registered for getting transaction details sent by the Store. The same event is also used to send restored purchases infomation. Now, you need to add the code to initiate restore request. And following code helps you do that:

private void RestoreCompletedTransactions ()
{
	NPBinding.Billing.RestoreCompletedTransactions ();
}
Unit Testing
When you test billing feature on a physical device, you are not interacting with Production servers (live servers), but instead your app will be using Sandbox servers. And that means you can test the features without fear of getting charged. 

In iOS device, for testing functionalities you need to setup Sandbox test user accounts in iTunes Connect and also ensure that you are logged out of the Store with real account.