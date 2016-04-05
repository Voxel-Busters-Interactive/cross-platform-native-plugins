# Address Book

Functionality
Access First Name, Last Name , Picture, Email Address and Phone Numbers from device's address book.

Usage
Get Authorization Status

NPBinding.AddressBook.GetAuthorizationStatus();
Read Contacts

Start reading contacts
/* Call for starting reading contacts. */
NPBinding.AddressBook.ReadContacts(OnReceivingContacts);
 

Receive data in callback   

/* Callback method which will be called after reading of contacts is done. */
void OnReceivingContacts(eABAuthorizationStatus _authStatus, AddressBookContact[] _contactList)
{
	if (_authStatus == eABAuthorizationStatus.AUTHORIZED)
	{
		//Start using _contactList here
	}
	else
	{
		Debug.LogError("[AddressBook] " + _authStatus);
	}
}
Use GetImageAsync method of AddressBookContact instance to fetch the image asyncronously.     

eABAuthorizationStatus is an enum which lists different states possible for an address book reading operation.

Editor Simulation
Add sample contact entries to test on editor. Editor inpector for Address Book is available at Window/VoxelBusters/NativePlugins/AddressBook.

