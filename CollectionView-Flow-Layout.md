## Configuring how the items in a `UICollectionView` are arranged (`UICollectionViewLayout`)

In the `UICollectionView` setup demo, we're left with gaps between each cell. In order to adjust that, we need to change the `UICollectionViewLayout`.

[`UICollectionViewLayout`][layoutlink] is where the `UICollectionView` defines how it will layout its data. `UICollectionView` are not constrained to a strict, single column, vertical format, unlike `UITableView`. `UICollectionView` can display items in any fashion, provided they are specified by a `UICollectionViewLayout`. In order to do this, a developer needs to subclass `UICollectionViewLayout` and specify the layout rules inside.

[layoutlink]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionViewLayout_class/index.html

### Flow Layout

By default, Apple provides developers with a subclass of `UICollectionViewLayout` called `UICollectionViewFlowLayout`. When you add a `UICollectionView` in the storyboard, it comes with a `UICollectionViewFlowLayout` already included.

[`UICollectionViewFlowLayout`][flowlayoutlink] is a layout that displays items in a grid format, with any number of cells in each section, and in vertical or horizontal directions. The items in a single section can be surrounded with header or footer views. It commonly covers most basic cases for the layout of `UICollectionView`.

[flowlayoutlink]:https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html

`UICollectionViewFlowLayout` can be configured in a number of ways:

* Specifying the size of items
* Specifying the space between items and lines
* Using section insets to adjust the margins of content

#### Modifying a Flow Layout
When adding a `UICollectionView` to a view controller in the storyboard, a `UICollectionViewCell` and `UICollectionViewFlowLayout` are included by default. You can modify the layout of your `UICollectionView` by:

* Editing attributes and dimensions of the Collection View Flow Layout in the storyboard
* Programmatically changing values and defining behavior

If your collection view only requires cells of the same static, non-proportional size, then use storyboard to adjust values. If you need a more varied layout, then see [customizing the cells and spacing](#customizing-the-cells-and-spacing-UICollectionViewDelegateFlowLayout)

#### Configuring the Flow Layout with the Storyboard
By default, a `UICollectionViewFlowLayout` is provided to you in the storyboard. If you select the Collection View Flow Layout, under your collection view, in the Utilities pane, you can alter attributes and dimensions.

#### Configuring the Flow Layout programmatically
In the code, you can configure the Flow Layout's properties and define rules for dimensions to change according to.

To access the properties of the layout of your `UICollectionView`, you can make an outlet for `UICollectionViewFlowLayout` like how you made one for `UICollectionView`. You can then directly set values, like in the storyboard.

```
class ViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate {
    @IBOutlet weak var collectionView: UICollectionView!
    @IBOutlet weak var flowLayout: UICollectionViewFlowLayout!
    
	override func viewDidLoad() {
        super.viewDidLoad()
        
        collectionView.delegate = self
        collectionView.dataSource = self

        flowLayout.scrollDirection = .Horizontal
        flowLayout.minimumLineSpacing = 0
        flowLayout.minimumInteritemSpacing = 0
        flowLayout.sectionInset = UIEdgeInsetsMake(0, 0, 0, 10)
    }
}
```

#### Customizing the cells and spacing (UICollectionViewDelegateFlowLayout)

If you need to customize cells and spacing, changing the direct properties of the Flow Layout will not suffice. You will need to implement the `UICollectionViewDelegateFlowLayout` protocol. With the `UICollectionViewDelegateFlowLayout` protocol, item sizes, section insets, line spacing and interitem spacing can all be made to change dynamically according to their position in the collection view.

<a href="https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html"><img src="https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Art/flow_horiz_layout_uneven_2x.png" title="Different sized cells" /></a>

To define dynamic rules to layout a UICollectionView, we add the [`UICollectionViewDelegateFlowLayout`][flowlayoutprotocollink] protocol.

[flowlayoutprotocollink]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionViewDelegateFlowLayout_protocol/index.html

```
class ViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate, UICollectionViewDelegateFlowLayout {
	...
}
```

#### Small Example of configuring the Flow Layout
Here is a small example of configuring the cell size according to the row. Calculating how large each cell should be, to have three cells per row, and changing the height depending on whether the row is even or odd.

<a href="http://imgur.com/ehSq2LS"><img src="http://i.imgur.com/ehSq2LS.png" title="Different sized cells example" /></a>

```
class ViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate, UICollectionViewDelegateFlowLayout {

    @IBOutlet weak var collectionView: UICollectionView!
    @IBOutlet weak var flowLayout: UICollectionViewFlowLayout!
    
    override func viewDidLoad() {
        super.viewDidLoad()

		...
        
        flowLayout.scrollDirection = .Horizontal
        flowLayout.minimumLineSpacing = 0
        flowLayout.minimumInteritemSpacing = 0
        flowLayout.sectionInset = UIEdgeInsetsMake(0, 0, 0, 0)
    }
	
	...
    
    // MARK: UICollectionViewDelegateFlowLayout methods 
    func collectionView(collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAtIndexPath indexPath: NSIndexPath) -> CGSize {
        let totalwidth = collectionView.bounds.size.width;
        let numberOfCellsPerRow = 3
        let oddEven = indexPath.row / numberOfCellsPerRow % 2
        let dimensions = CGFloat(Int(totalwidth) / numberOfCellsPerRow)
        if (oddEven == 0) {
            return CGSizeMake(dimensions, dimensions)
        } else {
            return CGSizeMake(dimensions, dimensions / 2)
        }
    }
```
#### Do you need a more custom layout?

You don't always want to use a `UICollectionViewFlowLayout` and sometimes, you may want something more customized.

Apple provides a [good guide][doyouneedacustomlayoutlink] on how to decide whether you need something more specific.

[doyouneedacustomlayoutlink]:https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html#//apple_ref/doc/uid/TP40012334-CH3-SW4

### Creating a custom UICollectionViewLayout
to be completed...