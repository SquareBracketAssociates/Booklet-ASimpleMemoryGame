## Basic building card graphical elements

In this chapter, we will build the visual appearance of the cards step by step.
In Bloc, visual objects are called elements, which are usually subclasses of `BlElement`, the inheritance tree root. Elements are the basic visual building blocks of Bloc. 
In subsequent chapters, we will add interaction using event listeners.



### First: the card element

Our graphic element representing a card will be a subclass of the `BlElement`. 
This element has, in addition, a reference to a card model (as defined in the previous chapter).

```
BlElement << #MGCardElement
	slots: { #card };
	tag: 'Elements';
	package: 'Bloc-Memory'
```



We define the corresponding accessors since the setter methods will be the place to hook registration for the communication 
between the model and the view, as we will show later.

```
MGCardElement >> card
	^ card
```

```
MGCardElement >> card: aMgCard
	card := aMgCard
```


The message `backgroundPaint` will be used later to customize the background of our card element.
Let us define a nice color. 

```
MGCardElement >> backgroundPaint
	"Return a BlPaint that should be used as a background (fill)
	of both back and face sides of the card. Colors are polymorphic
	with BlPaint and therefore can be used too."
	
	^ Color pink darker
```


We define a method `initialize` to set the size and the default color as well as a card model object.

```
MGCardElement >> initialize
	super initialize.
	self size: 80 @ 80.
	self background: self backgroundPaint.
	self card: (MGCard new symbol: $a)
```


### Starting to draw a card

In Bloc, `BlElement`s draw themselves onto the integrated canvas of the inspector as we inspect them, take a look at our element by executing this (See Figure *@InspectingPink0@*).

```
MGCardElement new inspect
```


![A first extremely basic representation of face down card.](figures/InspectingPink0.png width=60&label=InspectingPink0)

### Improving the card visual 


Instead of displaying a full rectangle, we want a better visual. 
Bloc lets us decide the geometry we want to give to our elements, it could be a circle, a triangle or a rounded rectangle for example, you can check available geometries by looking at subclasses of `BlElementGeometry`.
We can also add a png as we will show later.

We can start giving a circle shape to our element, we will need to use the `geometry:` message and give a `BlCircleGeometry` as an argument. You should obtain an inspector as shown in Figure *@InspectingCirclePink@*.

```
MGCardElement >> initialize
	super initialize.
	self size: 80 @ 80.
	self background: self backgroundPaint.
	self geometry: BlCircleGeometry new.
	self card: (MGCard new symbol: $a)
```


![A card with circular geometry.](figures/InspectingCirclePink.png width=60&label=InspectingCirclePink)

However, we don't want the card to be a circle either. 
We would like to have a rounded rectangle so we use the `BlRoundedRectangleGeometry` class.
We need to give the corner radius as a argument of the `cornerRadius:` class message. This is what we do in the following `initialize` method. 

```
MGCardElement >> initialize
	super initialize.
	self size: 80 @ 80.
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a)
```


You should get a visual representation close to the one shown in Figure *@figrounded@*.


![A rounded card.](figures/CardPinkRounded width=60&label=figrounded)


### Two faces of a card

A card has two faces: its back and its front. 
There are several approaches to manage this. 
In this tutorial, we will compose i.e, add/remove elements as children of the `MGCardElement` instance. 

- For the back we will add an element containing a png.
- For the front we will add a text element with a number.


### Defining some utilities
@utilitiesone

Since we do not want to duplicate size card logic, we define a simple method to return the extent of a card. 

```
MGCardElement >> cardExtent
	^ 80@80
```

And we use it to initialize the card element: 
```
MGCardElement >> initialize
	super initialize.
	self size: self cardExtent.
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a)
```

We could do the same for the corner radius but this is not important now. 

To manage the back of a card, we will read a png file and define it as a method to be able to version it. You can find the current definition on the class side of `MGCardElement`.

Since this is a large method that contains the textual serialization of a png we only show the beginning of its definition:

```
MGCardElement class >> cardbackForm
	^ Form
		extent: 80@80
		depth: 32
		bits: (Bitmap newFrom: #(16777215 16777215 16777215 16777215 16777215 16777215 16777215 16777215 318767103 1526726655 2936012799 3439329279 3942645759 4294967295 4294967295 4294967295 4294967295 
		...
```

You can read Section *@utilitiestwo@* to get more information about form and PNG handling. 


### Defining elements for card faces

We add two instance variables `backElement` and `frontElement` to refer to the children that  represent the contents of the two card faces. 


```
BlElement << #MGCardElement
	slots: { #card . #backElement . #frontElement };
	tag: 'Elements';
	package: 'Bloc-Memory'
```

We redefine the `initialize` method to create the `backElement` as well as adding a layout for 
placement of the children of the `MGCardElement` instances.
We use the method `asElement` on the cardback `Form` to quicly create a BlElement with the png as a background and the size equal to the png size.


```
MGCardElement >> initialize
	super initialize.
	backElement := self class cardbackForm asElement.			
	self size: self cardExtent.
	self layout: BlLinearLayout new alignCenter. 
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a).
```



In the following added part: 

```
backElement := self class cardbackForm asElement.	
frontElement := BlTextElement new.
```

We simply set a form as the background of this new element.
The method `cardbackForm` is the method illustrated above and that you can get loading the code of this tutorial.
If you want to create your own method, copy the logic of the method and paste the contents of the stream passed to the `storeOn:` method as in the following script:

```
String streamContents: [ :str | myForm storeOn: str ]
```

#### About layouts

In addition we initialized the layout to be a linear layout so that each child of our card element is centered. 

```
self layout: BlLinearLayout new alignCenter.
```

#### Better readability

We extract the back element creation in its own method `initializeBackElement`.

```
MGCardElement >> initializeBackElement
	backElement := self class cardbackForm asElement
```

```
MGCardElement >> initialize
	super initialize.
	self initializeBackElement.
	self size: self cardExtent.
	self layout: BlLinearLayout new alignCenter.
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a)
```


### Handling the front face

Now we will work on the front face visual.
First we will initialize a text element to a simple text element. 
This element will be updated for each card model. 

```
MGCardElement >> initializeFrontElement
	frontElement := BlTextElement new
```

```
MGCardElement >> initialize
	super initialize.
	self initializeBackElement.
	self initializeFrontElement.
	self size: self cardExtent.
	self layout: BlLinearLayout new alignCenter.
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a)
```

Second, we define the method `fillUpFrontElement` as follows: 


![Front face with a letter: inspect MGCardElement new.](figures/CardFrontManual.png width=60&label=CardFrontManual)


```
MGCardElement >> fillUpFrontElement
	frontElement text: (card symbol asString asRopedText
		fontSize: self fontPointSize;
		foreground: self fontColor;
		yourself)
```

To see the actual effect (See Figure *@CardFrontManual@*) we 
redefine the method `card:` as follows and inspect the result of `MGCardElement new`.

Let us explain it a bit, when a new card model is set, the text element representing the front
face is updated, the current children are emptied and the front element is added as child of the
card element. 

```
MGCardElement >> card: aCard
	card := aCard.
	self fillUpFrontElement.
	self removeChildren.
	self addChild: frontElement
```

We will use the same logic to switch back to the back face. 
We are now ready to implement the flipping of the card. 


### Flipping faces

Let us define two methods `showBackFace` and `showFrontFace` to encapsulate the 
logic of switching to a different face visual.

```
MGCardElement >> showBackFace
	self removeChildren.
	self addChild: backElement
```

```
MGCardElement >> showFrontFace
	self removeChildren.
	self addChild: frontElement
```

We redefine the method `card:` to put in place the corresponding visual based on the flipped status of the card model. 

```
MGCardElement >> card: aCard
	card := aCard.
	self fillUpFrontElement.
	card isFlipped
		ifTrue: [ self showFrontFace ]
		ifFalse: [ self showBackFace ].
```

We can do another step to factor nicely the behavior of the method `card:`.
We define a new method called `showCardFace`

```
MGCardElement >> showCardFace
	card isFlipped
		ifTrue: [ self showFrontFace ]
		ifFalse: [ self showBackFace ]
```

```
MGCardElement>> card: aCard
	"Attach a card model and subscribe to its announcements."
	
	card := aCard.
	self fillUpFrontElement.
	self showCardFace.
```

### From the model side 

Now we are ready to develop the flipped side of the card. To see if we should change the card model you can use the inspector to get the card element and send it the message `card flip` or directly 
recreate a new card  as follows: 

```
| cardElement | 
cardElement := MGCardElement new.
cardElement card flip.
cardElement
```


### Resources
@utilitiestwo

This section complements Section *@utilitiesone@*.
If you want to use your own pngs, have a look at the class `ReaderWriterPNG` that converts PNG files into Forms. A form is a piece of graphical memory internally used by Pharo.
So you have to convert your graphics from or to Forms. 

Here are some little scripts (that you should execute in order if you want to reproduce their effect.)

To save a form as a PNG on your disk:

```
PNGReadWriter putForm: MGCardElement cardbackForm onFileNamed: 'CardBack.png'
```

To save a form as a text (as shown above) that you can later execute to recreate the original form. 

```
| text | 
text := String streamContents: [ :str |
	(PNGReadWriter formFromFileNamed: 'CardBack.png') storeOn: str ].

"to recreate the form from its textual representation"
text := MGCardElement 
Object readFrom: text readStream 

```

#### Using Uuencoded strings

Storing a form in a plain text can produce large files, you can also use uuencoded of them.
This is what IconFactory project is doing. 

If you want to manage forms as the method cardbackForm provided in the project, you can have a look  at the IconFactory project on github. 

```
Metacello new
    baseline: #IconFactory;
    repository: 'github://pharo-graphics/IconFactory';
    load
``` 

This project supports the definition of form as textual resources in methods that can be then versioned altogether with the code. 

Given a base64 encoded string you can get a form with the following expression, here we take the base64 encoded string from `IconFactoryTest new exampleIconContents`

```
Form fromBinaryStream: IconFactoryTest new exampleIconContents base64Decoded asByteArray readStream
```

Following this you can generate a method body with a cache (here named icons) as follows: 

```
iconMethodTemplate
	^ '{1}
	"Private - Generated method"
	^ self icons
		at: #{1}
		ifAbsentPut: [ Form fromBinaryStream: IconFactoryTest new exampleIconContents
					 base64Decoded asByteArray readStream ]'
```
Where the first argument is part of a method name for example 'tintin'.

### Conclusion

We have all the visual elements for the card, so we are ready to work on the board game.
