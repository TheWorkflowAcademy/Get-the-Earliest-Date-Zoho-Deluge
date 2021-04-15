# Get-Earliest-Date-Zoho-Deluge
A Deluge script that performs date comparison with a simple loop to get the earliest date from a list of dates.

## Core Idea
Suppose you have a custom date field on the Accounts module of Zoho CRM called "First Deal Closing Date" that needs to display the closing date of the first Closed Won deal related to the Account. You need a script that iterates through all related Deals that are Closed Won, compare the closing dates, find the earliest closing date, then populate the date in that custom field.


## Tutorial
* Get all related Deals records to the Account.
* Set the *firstid* variable as an empty string "".
* Set the *datecomp* variable. The idea here is to set a future date that is definitely ahead of all Closed Won Deals. This is just to make sure that the first deal meets the date comparison criteria that we will set in the loop later.
* Iterate through each Deal record in a `loop`, filter out only the Closed Won Deals, and set the date comparison criteria.
  * `days360(a.get("Closing_Date"),datecomp) > 0` - if the date is earlier than `datecomp`, it will meet the criteria.
  * Upon meeting the criteria,
    * The Deal ID is assigned to the *firstid* variable.
    * The Closing Date of the Deal is assigned to the *datecomp* variable.
* After the iteration is complete, the *firstid* variable will no longer be an empty string and the *datecomp* variable will contain the earliest Closing Date among all the Deals.
* Now that we have what we need, we can update the *datecomp* variable into the "First Deal Closing Date" field in Accounts.

```javascript
allDeals = zoho.crm.getRelatedRecords("Deals","Accounts",accountid);
if (allDeals.size() > 0)
{
	// Counters
	firstid = "";
	datecomp = today.addYear(1); // Add as many years as necessary. We just want to make sure that the first deal enters the loop below.
	for each  a in allDeals
	{
		if(a.get("Stage") == "Closed Won")
		{
			if(days360(a.get("Closing_Date"),datecomp) > 0)
			{
				firstid = a.get("id");
				datecomp = a.get("Closing_Date");
			}
		}
	}
	if(firstid != "")
	{
		update = zoho.crm.updateRecord("Accounts",accountid,{"First_Deal_Closing_Date":datecomp});
		info update;
	}
}
```
