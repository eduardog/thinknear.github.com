---
layout: post
title: "Advanced Angular UI Router Part I: Nested Views"
date: 2015-01-07 13:18:03 -0800
comments: true
categories: 
---

At ThinkNear, we have an in-house administrative dashboard that our ad operations team uses to set up and manage ad campaigns.
The dashboard is an AngularJS frontend with a Ruby on Rails backend, with the
 [ui-router](https://github.com/angular-ui/ui-router) plugin for permalinks and navigation.
While ngNewsletter's [Diving deep into the AngularUI Router](http://www.ng-newsletter.com/posts/angular-ui-router.html) was a helpful primer, while creating the system we found it didn't go deep enough.
This post describes how we used ui-router's Nested Views feature to achieve the site layout we wanted.

<!-- more -->

### Background

Our object hierarchy goes Contract > Insertion Order > Campaign.
We have many Contracts, Contracts can have many Insertion Orders, and Insertion Orders can have many Campaigns.
Ad Ops often needed to reference one object while working on another.
They preferred to open multiple tabs or browser windows and look at them side-by-side.
They need an easy way to open multiple Contract tabs from the main list, and a good single-Contract view.
They wanted to be able to use the back button to go up the hierarchy.
And they wanted to be able to pass around short links to specific objects that loaded quickly.

Here is the page structure we came up with:

Contract List View:

    [Site Header]
      [Contract Section Header]
        [Contract1]
        [Contract2]
        …
    [Site Footer]
    
Contract Detail View:

    [Site Header]
      [Contract Section Header]
        [Contract 1 Header]
          [Back to List button]
        [Contract 1 Details]
          [Contract 1 Insertion Order 1]
          [Contract 1 Insertion Order 2]
            …
    [Site Footer]

Insertion Order Detail View:

    [Site Header]
      [Contract Section Header]
        [Insertion Order 1 Header]
          [Back to Contract button]
        [Insertion Order 1 Details]
          [Insertion Order 1 Campaign 1]
          [Insertion Order 1 Campaign 1]
          …
    [Site Footer]

Campaign Detail View:

    [Site Header]
      [Contract Section Header]
        [Campaign 1 Header]
          [Back to Insertion Order button]
        [Campaign 1 Details]
    [Site Footer]

And here's the URL structure for permalinks:

* /contracts - List
* /contracts/new - New contract
* /contracts/1 - Contract 1 details
* /contracts/insertion_orders/new - New insertion order
* /contracts/insertion_orders/1 - Insertion order 1 details
* /contracts/campaigns/new - New campaign
* /contracts/campaigns/1 - Campaign 1 details

Once states are set up, UI Router handles permalinks and back button support.
The tricky part was organizing the states to get both the permalinks and site layout we wanted.
The flat permalinks were important not only for URL length, but to improve load speed by not requesting all of an object's parents.

We tried nesting the states but using absolute URL patterns to flatten the URL structure:

    .state('contracts', { 
      url: "/contracts" 
    })
    .state('contracts.contract_selected', { 
      url: "/{contract_id:[0-9]+}"
    })
    .state('contracts.contract_selected.insertion_order_selected', { 
      url: "^/contracts/insertion_orders/{insertion_order_id:[0-9]+}"
      // Absolute URL with no contract ID
    })
    
Unfortunately, this doesn't work because UI router requires the `contract_id` be present for the parent state!
So we structured the states to mirror our permalink structure:

    .state('contracts', { 
      url: "/contracts" 
    })
    .state('contracts.contract_selected', { 
      url: "/{contract_id:[0-9]+}"
    })
    .state('contracts.insertion_order_selected', { 
      url: "/insertion_orders/{insertion_order_id:[0-9]+}"
    })

To get the site layout we wanted, we took advantage of UI Router's [named views](https://github.com/angular-ui/ui-router/wiki/Multiple-Named-Views).
Named views allow you to have DOM elements from child states replace DOM elements from parents. 
If you declare a state with the `templateUrl` and `controller` at the top level, 
it will implicitly insert it in an un-named `ui-view` element in the parent. 
But if you declare a `views` property in your state, you can name views, and specify where in the hierarchy they should be inserted.

### Ui-Router Templates and States

Here's the final structure of states and templates:

##### Main Site Template

This was a static template, no dynamic content.

    <div class="header">Site Header</div>
    <div ui-view></div>
    <div class="footer">Footer</div>

#### Contract Section

##### State

    .state('contracts', {
      url: "/contracts",
      abstract: true,
      views: {
        "": {
          controller: 'ContractSectionController',
          templateUrl: 'contracts/index.html'
        },
      },
    })
    
##### Template #####
	
	<div class="header">Contract Section Header</div>
	<div ui-view="main"></div>

#### Contract List

This is the state that shows a list of all contracts.

##### State

When a state's parent is abstract, setting the URL matcher to an empty string means that it will automatically go to this child state when you enter the URL for the parent. 
Set up like this, going to `http://root.com/#/contracts` will take you to the contracts.list state:

    .state('contracts.list', {
    	url: "",
    	views: {
    		"main@contracts": {
    			controller: 'ContractListController',
    			templateUrl: 'contracts/list.html'
    		}
    	}
    })

##### Template

	<div class="list">
		<div ng-repeat="item in list">
			<!-- list item content -->
		</div>
		<div class="pagination"></div>
	</div>


#### Contract Details

This is the state that shows a single contract.
It also functions as the permalink for that contract.

##### Single Item Template

Our single-item views had common elements, so we extracted those into a details template.

	<div class="detail-header">Item Header Content</div>
	<div ui-view="details"></div>

##### Contract Detail Template

This is shared by the existing item and new item states.

	<div class="contract">
		<!-- contract stuff goes here -->
	</div>
	
##### Contract Detail and New Contract State


Here you can see named views in action. 
`main@contracts` inserts the Single Item template into `<div ui-view="main">`, replacing the list. 
Then `details@contracts.contract_selected` inserts the Contract Detail template into `<div ui-view="details">`. 
The url matcher for `contract_selected` uses UI Router's [curly brace syntax](https://github.com/angular-ui/ui-router/wiki/URL-Routing#basic-parameters) to specify a regex for `contract_id`, to prevent it from matching on the `/new` we use for new contracts.

    .state('contracts.contract_selected', {
    	url: "/{contract_id:[0-9]+}",
    	views: {
    		"main@contracts": {
    			controller: 'SingleItemController',
    			templateUrl: 'contracts/single_item.html'
    		},
    		'details@contracts.contract_.selected', {
    			controller: 'ContractDetailController',
    			templateUrl: 'contracts/detail.html
    		}
    	}
    })
    
The new contract state is almost identical, but since we don't have an ID until the contract is saved, it matches on `/new`. 

    .state('contracts.contract_new', {
    	url: "/new",
    	…
    })

#### Insertion Order and Campaign Details

The Insertion Order and Campaign Detail states look just like Contract Detail State.

    .state('contracts.insertion_order_selected', {
    	url: "/{insertion_order_id:[0-9]+}",
    	views: {
    		"main@contracts": {
    			controller: 'SingleItemController',
    			templateUrl: 'contracts/single_item.html'
    		},
    		'details@contracts.insertion_order_selected', {
    			controller: 'InsertionOrderDetailController',
    			templateUrl: 'insertion_orders/detail.html
    		}
    	}
    })
    .state('contracts.campaign_selected', {
    	url: "/{campaign_id:[0-9]+}",
    	views: {
    		"main@contracts": {
    			controller: 'SingleItemController',
    			templateUrl: 'contracts/single_item.html'
    		},
    		'details@contracts.campaign_selected', {
    			controller: 'CampaignDetailController',
    			templateUrl: 'campaigns/detail.html
    		}
    	}
    })

### Conclusion and Next Steps

So that's how to use UI Router to display drill-down views for nested models.
Key takeaways:

* If you use state parameters, your state structure will need to mirror your permalink structure.
* Use named views to replace content from a parent state.

You may still have some questions, such as:

* How do you get the data into the controllers?
* Why isn't `contracts/single_item.html` part of an abstract parent state that the other detail views inherit from?

Those will be answered in the next part, UI Router Resolves.
