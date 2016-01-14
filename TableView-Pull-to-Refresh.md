##Adding a Pull-to-Refresh Control to your List (UITableView)

###Overview
Pull-to-refresh is a ubiquitous method to update your list with the latest information. Apple provides developers with [`UIRefreshControl`][uirefreshcontrol], as a standard way to implement this behavior. As `UIRefreshControl` is for lists, it can be used with `UITableView`, `UICollectionView`, and `UIScrollView`.

[uirefreshcontrol]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIRefreshControl_class/index.html

<a href="http://imgur.com/nVf3hiL"><img src="http://i.imgur.com/nVf3hiL.gif" title="source: imgur.com" /></a>


###Using a Basic UIRefreshControl
Using a basic `UIRefreshControl` requires the following steps:

1. Initialize an instance of UIRefreshControl
2. Implement an action to handle updating your list view
3. Bind the action to the instance of UIRefreshControl
4. Insert the UIRefreshControl as a subview of your list view

####Initializing a UIRefreshControl
In order to implement a `UIRefreshControl`, we need an instance of `UIRefreshControl`. Update the `viewDidLoad` function to include the following code:

```
override func viewDidLoad() {
	super.viewDidLoad()
	
	// Initialize a UIRefreshControl
	let refreshControl = UIRefreshControl()
}

```

####Implement an action to update the list
We need to implement an action for handling our list. It's common to fire a network request to get updated data, so below we're assuming that `fetchStories` exists. Make sure to call self.tableView.reloadData and refreshControl.endRefreshing after updating your data source.

```
func refreshControlAction(refreshControl: UIRefreshControlAction) {

	/*
		// Fetch latest data and in the callback:
			// Update data source with latest data
			// Call self.tableView.reloadData()
			// Call refreshControl.endRefreshing()
	*/
	
}
```

####Bind the action to the refresh control
With the action implemented, it now needs to be binded to the `UIRefreshControl` so that something will happen when you pull-to-refresh.

```
override func viewDidLoad() {
	super.viewDidLoad()
	
	// Initialize a UIRefreshControl
	let refreshControl = UIRefreshControl()
	refreshControl.addTarget(self, action: "refreshControlAction:", forControlEvents: UIControlEvents.ValueChanged)
}
```

####Insert the refresh control into the list
The `UIRefreshControl` now needs to be added to the list view.

```
override func viewDidLoad() {
	super.viewDidLoad()
	
	// Initialize a UIRefreshControl
	let refreshControl = UIRefreshControl()
	refreshControl.addTarget(self, action: "refreshControlAction:", forControlEvents: UIControlEvents.ValueChanged)
	tableView.insertSubview(refreshControl, atIndex: 0)
}
```

###Understanding a UIRefreshControl
The `UIRefreshControl` is a subclass of `UIView`, and consequently a view, so it can be added as a subview to another view. It's behavior and interaction is already built in. `UIRefreshControl` is a subclass of `UIControl`, and pulling down on the parent view will initiate a `UIControlEventValueChanged` event. When the event is triggered, a binded action will be fired. In this action, we update the data source and reset the `UIRefreshControl`. 

###Creating a custom pull-to-refresh
For now, see the following [reference][customRefreshControl]
[customRefreshControl]: http://www.jackrabbitmobile.com/design/ios-custom-pull-to-refresh-control/

###Gotchas
The following are common mistakes:

* After pulling to refresh, nothing changes? Did you forget `tableView.reloadData`?

- Is the UI changing in a strange sequence or the wrong order? Did you update the UI on the main thread (`dispatch_async(dispatch_get_main_queue())`)?

###Further notes
Apple documentation does specify that [`UIRefreshControl`][uirefreshcontrol] is only meant to be used with a `UITableViewController` where it is included, however we have not seen any problems in practice before.

>Because the refresh control is specifically designed for use in a table
>view that's managed by a table view controller, using it in a different
>context can result in undefined behavior.