# JensenCSETakeHomeChallenge

	Hello and welcome to my rudimentary Bay Area food truck location solution!  This solution was built entirely in Power Platform, so to try it out yourself, download either the managed or unmanaged solution archive files and import them to your own Power Platform environment.
	
	High-level solution architecture and explanations:
	
		○ Dataverse table to contain the list of current APPROVED food trucks
		    § I decided to copy the data out of the city's endpoint for scalability; if thousands of people end up using the app daily, the city might complain about live queries hammering their endpoints (I might be showing some bias from my experience with the OData endpoints in F&O).
			  § Loading only APPROVED trucks (as opposed to EXPIRED or REQUESTED) made sense functionally, and it also reduces the amount of irrelevant data being loaded to the Dataverse as well as being queried by the user
		○ PA Workflow to query the city's OData endpoint daily and completely regenerate the dataset in the Dataverse table.  I decided to go with a full drop/reload because 1) time is tight; and 2) there isn't that much data, especially when only loading approved trucks.
		○ PA Workflow to query the Dataverse food truck dataset for N food trucks near a given latitude and longitude.  If N or more trucks exist within a mile of your location, the workflow will return at least N food trucks to the caller.
		○ Two rudimentary Power Apps to validate the system: one which allows manual specification of lat/lon data as well as the desired number of results, and one which is closer to the "production" app, which uses the user's actual location and has a set requested result count of 5.
		○ There was only one requirement from the prompt, which I deviated from slightly by sometimes returning no results if there are no food trucks nearby.  This was an intentional design choice; I don't believe users will want to walk across the city for a food truck unless it is a truly mind-blowing gift to taste buds everywhere.

	Log of my thoughts as I built this:

		○ Reading through the prompt, I think in 3 hours I can build a PA Workflow (or Logic App, depending on which platform I have free credits on) to grab the top N closest food trucks to a lat/lon combination (5 for the prompt, but could change down the road) coupled with a very basic canvas-based Power App that allows the user to see the nearby food trucks for sure in a list, maybe on a map.
		    § Might need a second PA workflow to retrieve additional food truck details (i.e. FoodItems) once a user picks a location on the list/map
		○ To avoid any redundant storage I want to use the city's Odata endpoint "intelligently" to get the nearest food trucks, which seems tricky.  Given the relatively small range of latitude and longitude in San Francisco proper, the small size of the dataset, and the relative performance of the city's endpoint, I'll start with a one-second search radius and then double it as needed until I've got 5+ results.  Once the results are in the app, I can sort them by distance from the user's location on the fly.
		○ I definitely don't have free credits anywhere, guess I'm paying $15 for a month of PP licensing to use the premium features I need to finish this
		○ It occurs to me that using live web requests will not scale very well, it probably makes sense to avoid hammering the city's website by actually dumping the data into a dataverse table and querying that instead.
		    § Given the scalability problem plus problems I'm having with the Odata queries, I'm going to change course, load the data into a Dataverse table with a recurring update from the city's dataset, and use that data as the basis of my PA workflow.
		○ The Dataverse connector to the city's OData endpoint is problematic in that it really wants to use the HTTP with AAD connector, and said connector doesn't accept the reality that this connection does not need to be authenticated … so I'm making a new flow to manually update the dataverse daily (since the city doesn't seem consistent in when they publish the updated food truck list)
		○ It occurs to me that we probably don't want to show location data for trucks with expired/pending permits, so I'm omitting those from the result set
		○ I don't like making test-specific versions of programs, but I need to fudge lat/lon details to verify the lists are loading correctly, so I've made 2 separate versions of the app, one which actually reads the user's location, and the other which allows it to be input for testing.
		○ Ran out of time, but it would've been nice to:
		  § Sort the results by distance from the user
			§ Add a second page to display other details about the truck from the dataverse, since I'm capturing most of the metadata anyway
			§ Throw an Azure Maps map somewhere near the main screen
			§ Fix the weird dayshours null value on that seems somehow specific to approved food trucks
			§ Dump/clean the rest of the data problems in the city's current dataset (though all the 0/0 lat/lon combos proved super useful for testing!)
