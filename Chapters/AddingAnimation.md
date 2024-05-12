## Adding animation

In this chapter, we will add animations to the game. 
We will define animations and add them to the queue of card animation.
Then Bloc logic will execute the animation queue on the receiver.
This chapter will illustrate how animations can be composed or run in parallel.

### Card flipping animations

When the user flip the two cards we want to shrink them a bit to make them 
stand out. We use a simple scaling transformation.
An animation will modify the attributes (size, color, position...) of the element on which
it is applied.
Once the transformation is defined it is added to the animation queue of the 
receiver using the message `addAnimation:`. 

```
MGGameElement >> onFlippedFace
```

We define another similar animation to put back the full size of flipped back card. 

```
MGGameElement >> onFlippedBack
```

We modify the `showCardFace` method to invoke the method performing the animations
prior to changing the visual of the cards. 

```
MGGameElement >> showCardFace
```

### Card disappearing animation
When two cards match we want them to enlarge a bit to get player's attention.
When a card disappears, we will compose several animations: one that will grow
the element, another that will change its opacity and in parallel its size. 

The opacity animation will slowly make the card transparent while the size will make 
it is smaller.

We replace the previous `onDisappear` method by the following one. 

```
MGCardElement >> onDisappear
```

### Conclusion

This chapter presented how animations are simple to be defined in Bloc and how they are useful to enhance user experience. 