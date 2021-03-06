**SVGgh** *an SVG Rendering Framework for iOS*
-
Author [Glenn R. Howes](mailto:glenn@genhelp.com), *owner [Generally Helpful Software](http://genhelp.com)*

### Introduction
In my own apps, I've often wished to avoid using bitmapped images for my interface elements. Often, I'll need to add PNG files for Retina, and non-retina, iPhone and iPad, and find myself confined to what I can do with an interface in terms of stretching elements. And all this artwork made my app bulky. So, I decided to implement an SVG renderer which could use standard **Scalable Vector Graphics** documents to draw button icons, background art or whatever my art needs were. I have Apps in the App Store like [SVG Paths](http://AppStore.com/SVGPaths) whose only PNG files are the required icons. 

### Features
Handles shapes quite well such as paths, ellipses, circles, rectangles, polygons, polylines, and arcs with all the standard style attributes. Implements basic text and font handling, including rough text along a path. Implements both linear and radial gradients, including applying gradients to strokes and text. Implements scale invariant line widths. Provides a static UIView subclass, a UIControl button, and a segmented control. All are configurable from either nib or storyboards. Supports embedded bitmap images in standard formats. 

Can export to PDF, create UIImages, and print via the UIPrintInteractionController mechanism.

In the Xcode debugger, you can use the QuickLook button (the eye icon) to see the contents of an SVGRenderer. 

### Limitations
The entire [SVG specification](http://www.w3.org/TR/SVG11/) is not implemented. At present, it only implements the portions of the specification I needed or thought I might need. In particular, it doesn't support SVG fonts, animation, Javascript, or effects. Also, some attributes remain unimplemented or partially implemented, for example the *width* attribute of an *svg* entity cannot be expressed as a percentage. I hope users of this library will contribute back implementations of at least some of these. 

There are undoubtably bugs but I've used this library in all 8 apps I have in the App Store without issue so it is reasonably stable. Also, I would not label this a high performance renderer although I've never had cause to complain about it in the way I use it. 

The included library assumes ARC style memory management. It's also been arbitrarily set to support iOS 7 and up. I've moved to using newer code annotations such as *nullable* so it requires a recent version of Xcode to compile. Supports both traditional and module based framework includes.

Originally, this was distributed as a static library, but that is not a modern way to use it. So the enclosed project will build a framework, and most people will probably find the use of **Cocoapods** to be more enjoyable.

I've commented out **IB_DESIGNABLE** for the view classes pending more consistent and performant behavior on Xcode's part (or maybe I can figure out how to make it work better.) Try uncommenting it to see for yourself if it works, as it is satisfying when it does.

### As a Black Box Library
If you just want to use the code in your app and are uninterested in the underlying engine, the included Xcode project generates a framework (**SVGgh**) with the following public headers. By the way, the reason that classes tend to have a GH (Generally Helpful, or Glenn Howes) prefix is not narcissism, but an attempt of getting around the lack of a namespace in plain Objective-C.
* SVGDocumentView.h *A simple UIView capable of displaying a SVG document*
* GHButton.h *A flexible UIControl capable of having an embedded SVG document as an icon*
* GHSegmentedControl.h *A preliminary control which mimics a UISegmentedControl (incomplete)
* SVGParser.h *A class to load .svg files*
* SVGRenderer.h *A class to render SVG documents into a CGContextRef*
* GHControlFactory.h *A singleton class devoted to be a central location for widget theme look*
* GHImageCache.h *A singleton class devoted to caching and loading bitmap images*
* SVGToPDFConverter.h A class to convert the renderer's contents to a PDF.
* SVGPrinter.h A class to send a renderer's contents to a printer.

####If you are familiar with using Cocoapods and using it in your project
* Insert ````pod 'SVGgh'```` into your PodFile
* Go through the standard procedures for updating your Xcode workspace via Cocoapods. ````pod update````, ````pod install````, etc.

####If you are not using Cocoapods
To compile the framework. 
* Load the included **SVGgh.xcodeproj** project in Xcode 6.3 or above 
* **Build** the **Framework** target.
* Locate the **SVGgh.framework** framework
* Drag the framework into your own Xcode project

To use, you'll want to follow the following steps:
* Add the **SVGgh** library to your Xcode project.
* \#include &lt;SVGgh/SVGgh.h&gt;

####Once you have installed the library
* early in the launch of your app call 
    **[GHControlFactory kColorSchemeClear];**
* early in the launch of your app call
**MakeSureSVGghLinks();** in order to link classes only referenced in storyboards or nibs. As in:

````
#import <SVGgh/SVGgh.h>

@implementation YourAppDelegate

+(void) initialize
{
    [super initialize];
    MakeSureSVGghLinks(); // classes only used in Storyboards might not link otherwise
    [GHControlFactory setDefaultScheme:kColorSchemeClear];
    [GHControlFactory setDefaultTextColor:[UIColor greenColor]];
}
...
````

* If you are coding in Swift. You will want to add ````#import <SVGgh/SVGgh.h>```` to your bridging header. And in your App delegate you should probably put the initialize code somewhere early, like:

````
    override class func initialize()
    {
        super.initialize()        
        MakeSureSVGghLinks()
        let tintColor = UIColorFromSVGColorString("#5D6")!
        GHControlFactory.setDefaultButtonTint(tintColor)
    }
````

To add a button to a .xib file or storyboard:
* Drag a UIView into your view
* In the **Identity Inspector** give it a **Custom Class** name of **GHButton**
* In Xcode you should see the following custom attributes in the Attribute Inspector pane

| Key Path | Type | Value |
| -------- | ---- | ----- |
| Artwork Path | String | Artwork/MenuButton (assumes you have such an asset) |
| Scheme Number | Number | 3 |


* Note that the **.svg** extension is assumed
* or if you are making a text button:

| Key Path | Type | Value |
| -------- | ---- | ----- |
| Title | Localized String | My Label |
| Scheme Number | Number | 3 |

* Here is a listing of available color schemes, some of which are more useful than others. I prefer 3, kColorSchemeClear.

| Constant |     Enumeration     | Description                                 |
| :------: | :----------------- | --------------------------------------------|
|    0     |    kColorSchemeiOS  | Round solid buttons with a thin inset ring  |
|    1     | kColorSchemeMachine | Grey top to bottom gradient with inset ring |
|    2     | kColorSchemeKeyboard| Gray gradient, light on top, no ring        |
|    3     | kColorSchemeClear   | Gray gradient, light on top, ring           |
|    4     | kColorSchemeEmpty   | No chrome. Just the artwork or label        |
|    5     | kColorSchemeHomeTheatre | Garish gold gradient, ring              |
|    6     | kColorSchemeiOSVersionAppropriate | Same as kColorSchemeEmpty for now|
|    7     | kColorSchemeFlatAndBoxy | Solid fill color with square corners.   |

* There is an attribute of an SVG document called ````currentColor````. You can access it to change the appearance of a button while being pressed via the **textColor**, **textColorPressed** and **textColorSelected** properties of **UIControl**. These are accessible from storyboard or you can set it up globally in your initialize method. Your SVGs will have to be written to use currentColor instead of some explicit color.

To add a static view to a .xib file or storyboard:
* Drag a UIView into your view
* In the **Identity Inspector** give it a **Custom Class** name of **SVGDocumentView**
* Using Xcode 6 and above you should see the following custom attribute in the Attribute Inspector pane

| Key Path | Type | Value |
| -------- | ---- | ----- |
| Artwork Path | String | Artwork/MyBackground |

* Note that the **.svg** extension is assumed
* You should likely open the **Attributes Inspector** tab and set the **Mode** to **Aspect Fit** or possibly **Aspect Fill**.
	
#### Hints
* I like adding an Artwork folder to my target added as a 'Folder Reference' so that I can just drop things in from the Finder and they'll be added. Folder references show up in Xcode as blue folders.
* SVG has the ability to localize content as in the following fragment which localizes to Chinese:

```
<switch>
	<g systemLanguage ="zh">
		<text x="24" y="20" font-family="Helvetica" font-size="22"  fill="grey">绝对直线</text>
		<text x="218" y="95" font-family="Helvetica" font-style="italic" font-size="20" text-anchor="end" fill="blue">终点 x, y</text>
	</g>
	<g>
		<text x="24" y="20" font-family="Helvetica" font-size="22"  fill="grey">line to</text>
		<text x="218" y="95" font-family="Helvetica" font-style="italic" font-size="20" text-anchor="end" fill="blue">end x, y</text>
	</g>
</switch>
```	
* I've provided an Example.xcodeproj which displays an SVG in a view and displays a sharing button. 

### Under the Hood
If you are inclined to fix bugs or add features, and please do, then you'll be interested in the general mechanism by which an SVG document is converted to an onscreen image.     

The starting point is the **SVGRenderer**, which as a subclass of **SVGParser** is capable of loading in the XML of an SVG document and being used to render into a Core Graphics context (a CGContextRef).  The parser takes the XML and converts it to a tree composed of NSDictionaries and NSArrays of NSDictionaries. The NSDictionary at the root of this tree is used to create an **GHShapeGroup** which starts the process of building up a tree of **SVGAttributedObject**s, each of which knows how to render themselves. In general, when sending a message to an SVGAttributedObject, an object which implements the **SVGContext** protocol is provided so that certain state information is available to the SVGAttributedObject such as the currentColor or a method to look up the document tree for either a named object or an attribute defined by a parent. 

I've gone through and added Doxygen style comments to all the header files, so there is some hope of finding your way. 

### Attribution
While the vast majority of the code in this release was written by me. There are a couple of classes or categories that were found online but have a flexible enough license for me to include here.
* Jonathan Wight wrote a Base64 Transcoder which I found quite useful for handling embedded images.
* Ian Baird wrote a category for NSData for Base64 which I also found very easy to use. 
* [Ryan Hornberger] (http://www.ryanhornberger.com) was thoughtful enough to do something I had been too slammed to do: create a CocoaPod Spec for this library making it much more useful.
