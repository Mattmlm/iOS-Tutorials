##Adding Infinite Scroll Behavior to your List (UITableView)

###Overview
As a user is going through the list, they will eventually reach the end. When they reach the end, the service may still have more data for them. In order to provide the user with a smooth experience, and avoid pagination, we implement infinite scrolling to a list view.

Implementing infinite scroll behavior involves the following:

1. Detect when to request more data
2. Request more data and update UI accordingly
3. Create a progress indicator for user to know when data is being requested
4. Update progress to indicate with UI when data is being requested and loaded

Here is a sample image of what we're going to make:

<a href="http://imgur.com/KBSolR3"><img src="http://i.imgur.com/KBSolR3.gif" title="Infinite Scroll" /></a>

###Detecting when to request more data

####How to bind actions to changes in scrolling
In order to detect when to request more data, we need to use the `UIScrollViewDelegate`. `UITableView` is a subclass of `UIScrollView` and therefore can be treated as a `UIScrollView`. For `UIScrollView`, we use the `UIScrollViewDelegate` to perform actions when scrolling events occur. We want to detect when a `UIScrollView` has scrolled, so we use the method [`scrollViewDidScroll`][scrollviewdidscrolllink].

[scrollviewdidscrolllink]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIScrollViewDelegate_Protocol/#//apple_ref/occ/intfm/UIScrollViewDelegate/scrollViewDidScroll:

Add `UIScrollViewDelegate` to the protocol list and add the `scrollViewDidScroll` function:

```
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource, UIScrollViewDelegate {
	
	...
	
	func scrollViewDidScroll(scrollView: UIScrollView) {
		// Handle scroll behavior here
	}
}

```

####Understanding [`scrollViewDidScroll`][scrollviewdidscrolllink]
When a user scrolls down a list, the `UIScrollView` continuously fires `scrollViewDidScroll` after the `UIScrollView` has changed. This means that whatever code is inside `scrollViewDidScroll` will repeatedly fire. In order to avoid 10s or 100s of requests to the server, we need to indicate when the app has already made a request to the server.

Create a flag called `isMoreDataLoading`, and add a check in `scrollViewDidScroll`:

```
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource, UIScrollViewDelegate {
	
	var isMoreDataLoading = false
	
	func scrollViewDidScroll(scrollView: UIScrollView) {
		if (!isMoreDataLoading) {
			isMoreDataLoading = true
			
			// Code to load more results
			
			...
		}
	}
}
```

####Define the conditions for requesting more data
Ideally for infinite scrolling, we want to load more results before the user reaches the end of the existing content, and therefore would start loading at some point past halfway. However, for this specific tutorial, we will load more results when the user reaches near the end of the existing content.

We need to know how far down a user has scrolled in the `UITableView`. The property `contentOffset` of `UIScrollView` tells us how far down or to the right the user has scrolled in a `UIScrollView`. The property `contentSize` tells us how much total content there is in a `UIScrollView`. We will define a variable `scrollOffsetThreshold` for when we want to trigger requesting more data - one screen length before the end of the results.

```
func scrollViewDidScroll(scrollView: UIScrollView) {
	if (!isMoreDataLoading) {
		// Calculate the position of one screen length before the bottom of the results
		let scrollViewContentHeight = tableView.contentSize.height
		let scrollOffsetThreshold = scrollViewContentHeight -tableView.bounds.size.height
		
		// When the user has scrolled past the threshold, start requesting
		if(scrollView.contentOffset.y > scrollOffsetThreshold && tableView.dragging) {
		
			isMoreDataLoading = true
			
			// Code to load more results
			
			...
			
			
		}
	}
}
```

###Request more data
Define a separate function to request more data, and call this function in `scrollViewDidScroll`:

```
func loadMoreData() {

	/*
		// Fetch more data and in the callback:
			// Update flag
			self.isMoreDataLoading = false
			
			// Update data source with latest data
			self.data = data
			self.tableView.reloadData()
	*/

}

func scrollViewDidScroll(scrollView: UIScrollView) {
	if (!isMoreDataLoading) {
		// Calculate the position of one screen length before the bottom of the results
		let scrollViewContentHeight = tableView.contentSize.height
		let scrollOffsetThreshold = scrollViewContentHeight -tableView.bounds.size.height
		
		// When the user has scrolled past the threshold, start requesting
		if(scrollView.contentOffset.y > scrollOffsetThreshold && tableView.dragging) {
		
			isMoreDataLoading = true
			
			// Code to load more results
			loadMoreData()

		}
	}
}
```

When you run the app, you should see new data load into the list, each time you reach the bottom. But there's a problem: there's no indicator to the user that anything is happening.

###Creating a progress indicator
To indicate to the user that something is happening, we need to create a progress indicator.

There are a number of different ways a progress indicator can be added. Many cocoapods are available to provide this. However, for this guide, we'll create a custom view class and add it to the tableView's view hierarchy. Below is the code to create a custom view class that can be used as a progress indicator. 

We make use of Apple's [`UIActivityIndicatorView`][activityindicatorlink] to show a loading spinner, that can be switched on and off.

[activityindicatorlink]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIActivityIndicatorView_Class/

<img src="http://i.stack.imgur.com/9fmTG.png" title="Spinner" /></a>


```
class InfiniteScrollActivityView: UIView {
    var activityIndicatorView: UIActivityIndicatorView = UIActivityIndicatorView()
    static let defaultHeight:CGFloat = 60.0
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setupActivityIndicator()
    }
    
    override init(frame aRect: CGRect) {
        super.init(frame: aRect)
        setupActivityIndicator()
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        activityIndicatorView.center = CGPointMake(self.bounds.size.width/2, self.bounds.size.height/2)
    }
    
    func setupActivityIndicator() {
        activityIndicatorView.activityIndicatorViewStyle = .Gray
        activityIndicatorView.hidesWhenStopped = true
        self.addSubview(activityIndicatorView)
    }
    
    func stopAnimating() {
        self.activityIndicatorView.stopAnimating()
        self.hidden = true
    }
    
    func startAnimating() {
        self.hidden = false
        self.activityIndicatorView.startAnimating()
    }
}
```

###Update progress to indicate with UI when data is being requested
Now that we have a progress indicator, we need to load it when more data is being requested from the server.

####Add a loading view to your view controller
For this example, use the class `InfiniteScrollActivityView`, and add it to your view controller to use for later.

```
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource, UIScrollViewDelegate {
	
	var isMoreDataLoading = false
	var loadingMoreView:InfiniteScrollActivityView?

	...
}
```
In `viewDidLoad`, initialize and add the loading indicator to the tableView's view hierarchy. Then add new insets to allow room for seeing the loading indicator at the bottom of the tableView.

```
override func viewDidLoad() {
    super.viewDidLoad()

	// Set up Infinite Scroll loading indicator
	let frame = CGRectMake(0, tableView.contentSize.height, tableView.bounds.size.width, InfiniteScrollActivityView.defaultHeight)
	loadingMoreView = InfiniteScrollActivityView(frame: frame)
	loadingMoreView!.hidden = true
	tableView.addSubview(loadingMoreView!)
	
	var insets = tableView.contentInset;
	insets.bottom += InfiniteScrollActivityView.defaultHeight;
	tableView.contentInset = insets
}

```
Update the `scrollViewDidScroll` function to load the indicator:

```
func scrollViewDidScroll(scrollView: UIScrollView) {
	if (!isMoreDataLoading) {
		// Calculate the position of one screen length before the bottom of the results
		let scrollViewContentHeight = tableView.contentSize.height
		let scrollOffsetThreshold = scrollViewContentHeight -tableView.bounds.size.height
		
		// When the user has scrolled past the threshold, start requesting
		if(scrollView.contentOffset.y > scrollOffsetThreshold && tableView.dragging) {
		
			isMoreDataLoading = true
			                
            // Update position of loadingMoreView, and start loading indicator
            let frame = CGRectMake(0, tableView.contentSize.height, tableView.bounds.size.width, InfiniteScrollActivityView.defaultHeight)
            loadingMoreView?.frame = frame
            loadingMoreView!.startAnimating()

			// Code to load more results
			loadMoreData()
			
		}
	}
}
```

Finally, update the `loadMoreData` function to stop the indicator, when the request has finished downloading:

```
func loadMoreData() {

	/*
		// Fetch more data and in the callback:
			// Update flag and stop the loading indicator
			self.isMoreDataLoading = false
			self.loadingMoreView!.stopAnimating()
			
			// Update data source with latest data
			self.data = data
			self.tableView.reloadData()
	*/
}

```

###Credits
Thanks to [SVPullToRefresh][svpulltorefreshlink] for providing source code and a general flow to follow for this basic version of an infinite scroll.


###Using a cocoapod to implement instead
Infinite Scroll can also be implemented with the following cocoapod: [SVPullToRefresh][svpulltorefreshlink]

[svpulltorefreshlink]: https://github.com/samvermette/SVPullToRefresh

###Common Gotchas
To be added...