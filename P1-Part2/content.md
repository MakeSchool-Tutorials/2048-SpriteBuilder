---
title: Build your own 2048 with SpriteBuilder and Cocos2D - Part 2!
slug: part-2
---                       

[In the first part of this tutorial](https://www.makegameswith.us/gamernews/384/build-your-own-2048-with-spritebuilder-and-cocos2d) we set up the basic project, rendered a grid and implemented multiple methods to spawn random tiles when the game starts up. In this part we are going to add interaction to the game - we will start moving tiles!

# Move the tiles

We are now going to implement some exciting but also some slightly complicated functionality - tile movement. The basic rules of 2048 are:

*   the player can move all tiles in the grid in 4 different directions
*   each tile moves as far is it can in the direction the user chose
*   the tiles are moved in a specific order. Example: when moving to the left the left most tile in each row is moved first, then the second most left tile, etc.
*   tile movement is limited by the border of the grid and by other tiles. Tiles cannot move to grid slots that are already occupied by other tiles. The only exception is that two tiles can be merged when they have the same value.*   tiles can only be merged once per move. Tiles that already have been merged in a move cannot be merged again

We will not implement all of these rules at once. We will divide the functionality into smaller chunks. Once again it is very important to break the big problem down into smaller ones.

# Moving all tiles to the edges

In the first step we will be moving all tiles as far as possible in the selected direction, ignoring other tiles that might be in the way. To implement movement in general we will need to add some sort of input mechanism to the game.

## Adding gesture recognizers

For this game it will be most intuitive for the user to swipe in the direction in which the tiles shall move. Luckily iOS provides a very simple API to capture swipes. We can use the _UIGestureRecognizer._ Add these lines to the **end **of the _didLoadFromCCB _method:

        // listen for swipes to the left
        UISwipeGestureRecognizer * swipeLeft= [[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(swipeLeft)];
        swipeLeft.direction = UISwipeGestureRecognizerDirectionLeft;
        [[[CCDirector sharedDirector] view] addGestureRecognizer:swipeLeft];
        // listen for swipes to the right
        UISwipeGestureRecognizer * swipeRight= [[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(swipeRight)];
        swipeRight.direction = UISwipeGestureRecognizerDirectionRight;
        [[[CCDirector sharedDirector] view] addGestureRecognizer:swipeRight];
        // listen for swipes up
        UISwipeGestureRecognizer * swipeUp= [[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(swipeUp)];
        swipeUp.direction = UISwipeGestureRecognizerDirectionUp;
        [[[CCDirector sharedDirector] view] addGestureRecognizer:swipeUp];
        // listen for swipes down
        UISwipeGestureRecognizer * swipeDown= [[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(swipeDown)];
        swipeDown.direction = UISwipeGestureRecognizerDirectionDown;
        [[[CCDirector sharedDirector] view] addGestureRecognizer:swipeDown];

This code will add 4 listeners that listen for swipes in 4 different directions. Gesture recognizers need to be added to a _UIView_. The main _UIView_ in a Cocos2D application is the OpenGL view that is used to render the entire content of a Cocos2D app. We access this main _UIView_ through the _view_ property of _CCDirector_. The _UISwipeGestureRecognizer _allows to associate one method with each swipe direction. Input handling in iOS can be this simple!

Now we need to implement the methods that we have linked to our gesture recognizers. Add the following for methods to _Grid.m_:

    - (void)swipeLeft {
        CCLOG(@"swipeLeft");
    }
    - (void)swipeRight {
        CCLOG(@"swipeRight");
    }
    - (void)swipeDown {
        CCLOG(@"swipeDown");
    }
    - (void)swipeUp {
        CCLOG(@"swipeUp");
    }

For now we are just adding log statements to these methods. We want to be sure that the swipe detection works before moving forward.

Now you can run the app and check the console for log messages to appear. If you swipe in any direction you should see one of the four log messages in the Xcode console.

Now that this works as expected we can replace our test implementation with some actual code. We will not implement the tile movement directly in the gesture recognizer callbacks since that would result in a lot of duplicate and badly reusable code. Basically all of the methods would have the same code, the only difference would be the direction of the move. Instead we will choose an elegant solution and will have the gesture recognizer callbacks call a _move_ method and providing the direction in which the tiles should move.

To express the direction in which the tiles shall move we will use _CGPoint. _A _CGPoint _ is ideal to express a movement vector because it has an X and Y component. A movement to the left for example can be expressed as (-1,0).

Change the implementation of the swipe callback methods to call the move methods and pass the according vector as a parameter:

    - (void)swipeLeft {
        [self move:ccp(-1, 0)];
    }
    - (void)swipeRight {
        [self move:ccp(1, 0)];
    }
    - (void)swipeDown {
        [self move:ccp(0, -1)];
    }
    - (void)swipeUp {
        [self move:ccp(0, 1)];
    }

Great! Now we need to implement the _move _method that we are calling here. The _move_ method is the most complicated part of the _2048 _game so grab your favorite hot drink. We're going to dive right into it!

## Implementing the move method

This is a good time to do some recap on the rules that determine the tile movement in _2048_. The most important one is the order in which the tiles are moved. Here's an illustrated example of a movement to the left, that shows how each individual tile is moved:

![](https://static.makegameswith.us/gamernews_images/2Ua8UeudDO/TileMovement.png)

When moving to the left, the left most tile is moved first. When moving to the right the right most tile is moved first, etc. Then the second most left/right tile is moved, etc. This works the same way for moving tiles up or down. In this case the tile with number "2" is moved first. Since there are no tiles in the way it is moved entirely to the left side. In the second part of the movement tile "4" is moved. It cannot be moved entirely to the left because tile "2" occupies this spot. That's why it is moved to the second most left spot. _Note: the player will never see the intermediate step (Move Part 1) of the tile movement it is only illustrated here to explain the rules of 2048._

These rules basically mean that implementing tile movement is a three step process:

1.  Select the tile that needs be moved
2.  Determine how far this tile can be moved
3.  Repeat 1 and 2 until each tile in the game has been selected

It is fairly tricky to come up with a generic solution for this. We need a method that correctly determines in which order the tiles need to be moved, depending on which of the four directions have been selected.

I will provide this first version of the move method for you and walk through it line for line explaining the details. Add the following _move_ method to _Grid.m_:

    - (void)move:(CGPoint)direction {
        // apply negative vector until reaching boundary, this way we get the tile that is the furthest away
        //bottom left corner
        NSInteger currentX = 0;
        NSInteger currentY = 0;
        // Move to relevant edge by applying direction until reaching border
        while ([self indexValid:currentX y:currentY]) {
            CGFloat newX = currentX + direction.x;
            CGFloat newY = currentY + direction.y;
            if ([self indexValid:newX y:newY]) {
                currentX = newX;
                currentY = newY;
            } else {
                break;
            }
        }
        // store initial row value to reset after completing each column
        NSInteger initialY = currentY;
        // define changing of x and y value (moving left, up, down or right?)
        NSInteger xChange = -direction.x;
        NSInteger yChange = -direction.y;
        if (xChange == 0) {
            xChange = 1;
        }
        if (yChange == 0) {
            yChange = 1;
        }
        // visit column for column
        while ([self indexValid:currentX y:currentY]) {
            while ([self indexValid:currentX y:currentY]) {
                // get tile at current index
                Tile *tile = _gridArray[currentX][currentY];
                if ([tile isEqual:_noTile]) {
                    // if there is no tile at this index -> skip
                    currentY += yChange;
                    continue;
                }
                // store index in temp variables to change them and store new location of this tile
                NSInteger newX = currentX;
                NSInteger newY = currentY;
                /* find the farthest position by iterating in direction of the vector until we reach border of grid or an occupied cell*/
                while ([self indexValid:newX+direction.x y:newY+direction.y]) {
                    newX += direction.x;
                    newY += direction.y;
                }
                if (newX != currentX || newY !=currentY) {
                    [self moveTile:tile fromIndex:currentX oldY:currentY newX:newX newY:newY];
                }
                // move further in this column
                currentY += yChange;
            }
            // move to the next column, start at the inital row
            currentX += xChange;
            currentY = initialY;
        }
    }

That's a lot of code to digest, but basically it just implements the movement pattern that we have discussed earlier - let's take a close look at it.

The _move_ method get's the movement direction as a _CGPoint._

We start the method with searching for the tile that should be moved first. We start at index (0,0), which is the bottom left corner of the grid. We then move into the direction of the movement vector. We keep on moving into the direction of the movement vector until we hit an invalid index. We check that by using the _indexValid_ method which we are going to write later on. For example, when the move method would be called with a top direction, we would first move entirely to the top of the grid and start searching for the first tile to move from there. We store the current position on the grid in the _currentX _and _currentY _variables, these two variables basically have the function of a cursor. After completing this first step, _currentX _and _currentY _contain the position on the grid from which we will start searching for tiles to move.

From this initial position we search column for column for the first tile to move. We start at _currentY_ in each column, which is either the top or bottom edge of the grid, depending on the movement direction. Inside each column we move into the opposite y direction of the movement vector. Once we completed a column, we will move to the next column that is in the opposite x direction of the movement vector.

We capture the initial y position for each column in the variable _initialY. _We store the x and y movement direction in the variables _xChange _and _yChange_. These variables describe in which direction we move over the grid. Since the movement direction will always only be either in x or in y direction, either x or y will be 0. For example when moving to the left, the _yChange_ will be 0 because the y component of the movement direction is 0. We iterate through the grid with the inverted movement direction. However, it is only important to invert the movement direction for the part of the direction that is **not 0. **So for a movement to the left it is important that we select tiles beginning at the left corner and moving to the right. However, it is irrelevant if we start in the top row and move to the bottom or do it the other way round. However, _xChange _and _yChange_ are both not allowed to be 0, because we use them to iterate through our grid. So whichever of these two values is 0 will be set to 1. This way we ensure that we iterate through all rows and columns.

When wrapping your head around this the first time this can be very difficult to understand, so we have added an illustration to explain the concept further.

The following figure illustrates how we find the start position from which we start iterating through the tiles and how exactly we iterate through the tiles.

The two top images show how the first loop looks for where in the grid it should start selecting tiles.

The bottom image illustrates how the second loop iterates through all tiles from the starting position determined by the first loop.

For this example we assume a movement to the right:

![](https://static.makegameswith.us/gamernews_images/L7QIzfnKKh/TileSearch.png)

Now you should understand:

*   how we find the start position for selecting tiles
*   in which order we iterate through all the tiles of the grid

The last open question is, **how is the movement of each individual tile implemented?**

This happens in the most inner _while_ loop of the _move _method. Once we have selected an index on the grid and ensured that we have a tile stored at that index, we move that tile as far as possible. We apply the movement vector on the selected tile until we reach an invalid index. This is the section of the move method that implements the actual tile movement:

    ...
                // get tile at current index
                Tile *tile = _gridArray[currentX][currentY];
                if ([tile isEqual:_noTile]) {
                    // if there is no tile at this index -> skip
                    currentY += yChange;
                    continue;
                }
                // store index in temp variables to change them and store new location of this tile
                NSInteger newX = currentX;
                NSInteger newY = currentY;
                /* find the farthest position by iterating in direction of the vector until we reach border of grid or an occupied cell*/
                while ([self indexValid:newX+direction.x y:newY+direction.y]) {
                    newX += direction.x;
                    newY += direction.y;
                }
                if (newX != currentX || newY !=currentY) {
                    [self moveTile:tile fromIndex:currentX oldY:currentY newX:newX newY:newY];
                }
    ...

As you can see we have a _while_ loop that moves the selected tile as far as possible in the direction of the movement vector. It terminates once the tile reaches one of the edges of the grid.

After the loop terminates we check if the position of the selected tile has changed. For example, if the selected tile already is located left edge of the grid and we want to move it to the left, the position will not change. Only if the position changed we call the _moveTile_ method. The _moveTile_ method will update the position of the tile in the __gridArray_ and move the tile visually with an animation.

Quite a lot of code but basically this is the three step process described earlier:

1.  Select the tile that needs be moved next
2.  Determine how far this tile can be moved
3.  Repeat 1 and 2 until each tile in the game has been selected

Now all that is left to do is adding the _moveTile _and _indexValid_ methods that we are using in our implementation of the _move_ method.

## Adding the moveTile and indexValid method

Good news: the methods we are going to add in this step are lot shorter than the _move _method.

Let's start with the _indexValid_ method. The _indexValid_ method will receive a index position and will return a _BOOL_ value that describes wether the provided index is valid (within the grid) or not.

Add the _indexValid _method to _Grid.m_:

    - (BOOL)indexValid:(NSInteger)x y:(NSInteger)y {
        BOOL indexValid = TRUE;
        indexValid &= x >= 0;
        indexValid &= y >= 0;
        if (indexValid) {
            indexValid &= x < (int) [_gridArray count];
            if (indexValid) {
                indexValid &= y < (int) [(NSMutableArray*) _gridArray[x] count];
            }
        }
        return indexValid;
    }

All this method does is checking wether the index is within the bounds of the two dimensional array.

Now you need to add the _moveTile _method to _Grid.m_:

    - (void)moveTile:(Tile *)tile fromIndex:(NSInteger)oldX oldY:(NSInteger)oldY newX:(NSInteger)newX newY:(NSInteger)newY {
        _gridArray[newX][newY] = _gridArray[oldX][oldY];
        _gridArray[oldX][oldY] = _noTile;
        CGPoint newPosition = [self positionForColumn:newX row:newY];
        CCActionMoveTo *moveTo = [CCActionMoveTo actionWithDuration:0.2f position:newPosition];
        [tile runAction:moveTo];
    }

The _moveTile_ method receives the tile that should be moved, the old index, and the new index as method parameters. The first thing we do in this method is changing the position of the tile in the __gridArray_. We add the tile to the new index and remove the tile from the old index.

Then we take care of updating the UI so that a player sees the tile moving across the grid. Here we use the _positionForColumn_ method to get the new x and y coordinates for this tile on the grid. We then use a _CCActionMoveTo_ to animate the movement of the tile to that new position.

You have completed a huge amount of steps now:

*   Added gesture recognizers
*   Added gesture recognizer callbacks
*   Added a complex _move_ method
*   Added a _indexValid _method
*   Added a _moveTile _method

Now it's once again time to run the game and take a look at the results. You should be able to move the tiles from one edge to another by swiping in that direction:

![](https://static.makegameswith.us/gamernews_images/xLbBLaomVz/SwipeMove.gif)

**Well done! **Now the game already looks a lot like 2048. As you can see in the little demo above, tiles can overlap at the moment. Fixing this is the next little challenge we are going to tackle!

# <span style="">Avoid that tiles overlap</span>

Our current mechanism moves each tile in the direction of the movement until it reaches an invalid index. What the original game does is moving each tile until it reaches an invalid index **or **an occupied tile. This means we need an extension of our test in the _indexValid_ method.

Instead of changing the _indexValid_ method we will be adding a new method called _indexValidAndUnoccupied. _The _indexValid _method is used in multiple places that only need to check if a value is within the boundaries of the __gridArray _and that do not care about occupied or unoccupied cells, so we need to keep that method.

Add the new _indexValidAndUnoccupied_ method to _Grid.m_:

    - (BOOL)indexValidAndUnoccupied:(NSInteger)x y:(NSInteger)y {
        BOOL indexValid = [self indexValid:x y:y];
        if (!indexValid) {
            return FALSE;
        }
        BOOL unoccupied = [_gridArray[x][y] isEqual:_noTile];
        return unoccupied;
    }

This method receives an index position. It uses the _indexValid _method to check if the index is within the bounds of the __gridArray._ If that is the case it additionally checks if the provided index is occupied or unoccupied.

Now all we need to do is use this method when we check how far we can move a tile instead of  using the _indexValid_ method.

Modify this part of the move method:

    ...
                /* find the farthest position by iterating in direction of the vector until we reach border of grid or an occupied cell*/
                while ([self indexValid:newX+direction.x y:newY+direction.y]) {
                    newX += direction.x;
                    newY += direction.y;
                }
    ...

	To use our new method:

    ...
                /* find the farthest position by iterating in direction of the vector until we reach border of grid or an occupied cell*/
                while ([self indexValidAndUnoccupied:newX+direction.x y:newY+direction.y]) {
                    newX += direction.x;
                    newY += direction.y;
                }
    ...

Now you can run the game again and you should see that tiles cannot overlap anymore!

# Merge tiles

Now we have avoided that tiles overlap in the game but there is a special situation in _2048_ when tiles can actually merge. If two tiles have the same value they will merge into one. In this step we will add that functionality to our game.

The first part in adding this feature is extending our _Tile _class. It currently does not store any value yet, we need to add a property to capture the current value of the tile. We also need to add a method that can be called to update the displayed value of the _Tile_. As you will see later we will have situations where we need to change the value of a tile before we want the tile to display that new value, so we cannot refresh the displayed value automatically.

Open _Tile.h _and add the following property and the following method definition:

    @property (nonatomic, assign) NSInteger value;
    - (void)updateValueDisplay;

	Next we will add a couple of methods to _Tile.m_. When a tile gets initialized we need to assign a value to the new tile. In _2048_ each tile is spawned with a value of 4 or 2. Add the following init method to _Tile.m_:

    - (id)init {
      self = [super init];
      if (self) {
        self.value = (arc4random()%2+1)*2;
      }
      return self;
    }

This is a very simple implementation even though the part that generates the random number may look a little cryptic. We generate a random number that is either 2 or 4 and store it in the _value_ property.

Next, we need to add the _updateValueDisplay _method. Add the method to _Tile.m_:

    - (void)updateValueDisplay {
        _valueLabel.string = [NSString stringWithFormat:@"%d", self.value];
    }

This method  just updates the text of the label with the current value of the tile.

              As a last step we need to implement the _didLoadFromCCB_ method in _Tile.m_. Once the CCB file is loaded entirely we want our label to display the current value of the tile. Add the following implementation to _Tile.m_

    - (void)didLoadFromCCB {
        [self updateValueDisplay];
    }

Well done! Now you should see the initial tiles spawning with values of 4 or 2:

![](https://static.makegameswith.us/gamernews_images/Ji422fvMTB/iOS Simulator Screen shot 09 Apr 2014 17.19.20.png)

Now that tiles have values we are able to check if two tiles could be merged or not. We will need to add this check in our _move_ method.

Currently we move the tile in the selected direction until we reach an occupied or invalid index. Now, we need to add a check to see if we stopped moving further because of an occupied index. If that was the case we need to check if the tile that is blocking the index has the same value as our tile. If both have the same value we need to merge them, if they don't have the same value we move the tile next to the tile that is occupying the next index (this is the current default behaviour).

Replace this block inside the _move_ method:

    ...
                if (newX != currentX || newY !=currentY) {
                    [self moveTile:tile fromIndex:currentX oldY:currentY newX:newX newY:newY];
                }
    ...

	With this one:

                BOOL performMove = FALSE;
                /* If we stopped moving in vector direction, but next index in vector direction is valid, this means the cell is occupied. Let's check if we can merge them*/
                if ([self indexValid:newX+direction.x y:newY+direction.y]) {
                    // get the other tile
                    NSInteger otherTileX = newX + direction.x;
                    NSInteger otherTileY = newY + direction.y;
                    Tile *otherTile = _gridArray[otherTileX][otherTileY];
                    // compare value of other tile and also check if the other thile has been merged this round
                    if (tile.value == otherTile.value) {
                        // merge tiles
                        [self mergeTileAtIndex:currentX y:currentY withTileAtIndex:otherTileX y:otherTileY];
                    } else {
                        // we cannot merge so we want to perform a move
                        performMove = TRUE;
                    }
                } else {
                    // we cannot merge so we want to perform a move
                    performMove = TRUE;
                }
                if (performMove) {
                    // Move tile to furthest position
                    if (newX != currentX || newY !=currentY) {
                        // only move tile if position changed
                        [self moveTile:tile fromIndex:currentX oldY:currentY newX:newX newY:newY];
                    }
                }

As mentioned above, if we stop moving further because of an occupied index we dive into some additional investigation. We can detect a situation where the movement was stopped by another tile by using the _indexValid _and the _indexValidAndUnoccupied _methods. When _indexValidAndUnoccupied _returns _FALSE _and _indexValid_ returns _TRUE_ for the same index, we know that we have provided a valid index that is occupied. We can then check if the current tile can be merged with the occupying tile. If yes, we call the _mergeTileAtIndex_ method that we are going to implement next. If not, we set the _performMove_ variable to _TRUE_ to indicate that we want to perform a regular move.

Now let's implement the _mergeTileAtIndex_ method in _Grid.m_:

    - (void)mergeTileAtIndex:(NSInteger)x y:(NSInteger)y withTileAtIndex:(NSInteger)xOtherTile y:(NSInteger)yOtherTile {
        // 1) update the game data
        Tile *mergedTile = _gridArray[x][y];
        Tile *otherTile = _gridArray[xOtherTile][yOtherTile];
        otherTile.value *= 2;
        _gridArray[x][y] = _noTile;
        // 2) update the UI
        CGPoint otherTilePosition = [self positionForColumn:xOtherTile row:yOtherTile];
        CCActionMoveTo *moveTo = [CCActionMoveTo actionWithDuration:0.2f position:otherTilePosition];
        CCActionRemove *remove = [CCActionRemove action];
        CCActionCallBlock *mergeTile = [CCActionCallBlock actionWithBlock:^{
            [otherTile updateValueDisplay];
        }];
        CCActionSequence *sequence = [CCActionSequence actionWithArray:@[moveTo, mergeTile, remove]];
        [mergedTile runAction:sequence];
    }

Basically the _mergeTileAtIndex _method is very similar to the _moveTile _method. We first update the data model then we update the UI.

At the beginning of the method we get the _mergedTile _(the tile that is going to disappear after the merge) and the _otherTile, _the tile which's value will increase and that will "swallow" the _mergedTile. _We update the value of the _otherTile, _then we remove the _mergedTile _from the __gridArray._ Now all the game data is up to date.

In the second part of the method we take care of updating the visuals of the game. We create an animation sequence with three steps:

1.  Move _mergedTile _onto _otherTile_
2.  Increase displayed value of _otherTile_
3.  Remove _mergedTile_

**We have implemented a basic merge mechanism!**

Now you will be able to merge two tiles with the same value. Since we currently only have two initial tiles you might need to start the game a couple of times to get two tiles with the same value.

Once you got that the merging should look like this:

![](https://static.makegameswith.us/gamernews_images/TqA9q5cKGD/Merge.gif)

# Spawn new tiles each round

<span style="">Slowly it's time to add more tiles to the game. In _2048_ a new tile gets spawned whenever the player performs a move. An action in the game is only considered a "move" when one of the tiles actually changes positions. If the user chooses a direction that will not allow any tile on the grid to move then no new tile will be spawned.</span>

Since a new tile needs to be spawned upon each completed move, this functionality needs to be added to the _move _method. We will also need to introduce a variable that stores wether any of the tiles has moved in the current move action or not.

Add this variable definition to the beginning of the _move_ method:

       BOOL movedTilesThisRound = FALSE;

	Next, we need to set this variable to _TRUE_ when we moved or merged a tile. Update the following part of the _move_ method and add the lines that set _movedTilesThisRound_ to _TRUE_:

                if ([self indexValid:newX+direction.x y:newY+direction.y]) {
                    // get the other tile
                    NSInteger otherTileX = newX + direction.x;
                    NSInteger otherTileY = newY + direction.y;
                    Tile *otherTile = _gridArray[otherTileX][otherTileY];
                    // compare value of other tile and also check if the other thile has been merged this round
                    if (tile.value == otherTile.value) {
                        // merge tiles
                        [self mergeTileAtIndex:currentX y:currentY withTileAtIndex:otherTileX y:otherTileY];
                        movedTilesThisRound = TRUE;
                    } else {
                        // we cannot merge so we want to perform a move
                        performMove = TRUE;
                    }
                } else {
                    // we cannot merge so we want to perform a move
                    performMove = TRUE;
                }
                if (performMove) {
                    // Move tile to furthest position
                    if (newX != currentX || newY !=currentY) {
                        // only move tile if position changed
                        [self moveTile:tile fromIndex:currentX oldY:currentY newX:newX newY:newY];
                        movedTilesThisRound = TRUE;
                    }
                }

**Pay close attention to the changes above! **We have added **two lines** that set _movedTilesThisRound_ to _TRUE_.

Now there's only one change to the _move_ method left. Add the following lines to the end of the _move _method, **after all the loops have completed**:

        if (movedTilesThisRound) {
            [self spawnRandomTile];
        }

Now you will spawn a new tile whenever the existing tiles have been moved or merged! **Play the new version of the game to test this feature!** After a while your grid should look similar to this:

![](https://static.makegameswith.us/gamernews_images/QHJt1xYRBm/iOS Simulator Screen shot 09 Apr 2014 18.02.24.png)

**This is basically a playable game already, well done! **If you played long enough you will have realized that there is a little issue with the game. Unlike the original _2048_ a tile that has been merged in this move can be merged again! We are going to fix this in the next step.

# Avoid that tiles can merge twice

The rules of _2048 _don't allow a tile to merge twice within in one move. Here is an illustration of an example situation assuming an upward tile movement:

![](https://static.makegameswith.us/gamernews_images/lpHHJLb42W/TileMerge.png)

At the top you can see the example scenario. On the bottom left you can see what is happening in our version of the game. The two "4" tiles merge to an "8" tile and then the merged "8" tile merges with the other "8" tile to a "16" tile. This shouldn't happen. On the right you can see the expected outcome. The merged "8" tile cannot merge with any other tile, because it already has been merged in this move, so two "8" tiles remain on the grid.

To implement this we will need to add a _BOOL_ variable to our _Tile_ that will remember if a _Tile _has already been merged in a move. Open _Tile.h_ and add this property:

    @property (nonatomic, assign) BOOL mergedThisRound;

	Now we have a property that allows us to store if a tile has been merged in a move or not. We will need to use this property within our _mergeTileAtIndex_ and _move_ methods. Add this line to _mergeTileAtIndex_ after a value has been assigned to _otherTile:_

    otherTile.mergedThisRound = TRUE;

Now, whenever a tile gets merged, we set the flag to _TRUE_. Remember, the _otherTile_ is the tile that remains in the game and which's value is doubled.

Now that we know when a tile has been merged, we need to use that information in our _move_ method. Currently our _move_ method is responsible for starting a merge between two tiles.

Within the _move_ method we perform this check to to determine if two tiles can be merged:

    ...
    if (tile.value == otherTile.value) {
    ...

	Now we need to extend this check. Tiles should only be merged when they have the same value **and** haven't been merged this move yet. Change the line above to look like this:

    ...
    if (tile.value == otherTile.value && !otherTile.mergedThisRound) {
    ...

Great! Now we are correctly flagging tiles that already have been merged and checking for that flag when we are about to merge two tiles.

**What is the next step? What is missing to make the feature we just added work as expected?** Correct! We need to reset the _mergedThisRound_ flag after each move. We only forbid multiple merges of one tile within one move, however, when the move is completed all tiles on the grid can be merged again, thus we need to reset the value of _mergedThisRound_ for each tile in the game.

We are going to encapsulate the resetting of the _mergedThisRound_ flag in a method called _nextRound_, because we will have a couple of tasks that need to be performed after a move completed.

Inside our _move_ method we already have a great position to add this functionality. Change these lines inside the _move_ method:

         if (movedTilesThisRound) {
             [self spawnRandomTile];
         }

	To look like this:

         if (movedTilesThisRound) {
             [self nextRound];
         }

We already had defined that a round is completed if at least one tile moved in a  round. So far the only action we were undertaking when a round completed was spawning a new tile. Now that we also want to reset the _mergedThisRound_ flag it makes sense to extract all of this into a _nextRound_ method.

Now all we need to do is add the _nextRound_ method to _Grid.m_:

    - (void)nextRound {
        [self spawnRandomTile];
        for (int i = 0; i < GRID_SIZE; i++) {
            for (int j = 0; j < GRID_SIZE; j++) {
                Tile *tile = _gridArray[i][j];
                if (![tile isEqual:_noTile]) {
                    // reset merged flag
                    tile.mergedThisRound = FALSE;
                }
            }
        }
    }

Right at the beginning of the method we spawn a new random tile - that is the functionality that we moved from the _move_ method to this method.

The bottom part of the method iterates through all tiles in the grid and resets the value of the _mergedThisRound_ property.

**Well done!** Now we are actually really close to finishing the game. In this step we have added user input and implemented all the different rules for tile movement &amp; tile merging. All the basic game mechanics are implemented.

[Go to part 3 where we will add scores, win &amp; lose conditions and will apply some finishing touches to this game!](https://www.makegameswith.us/gamernews/388/build-your-own-2048-with-spritebuilder-and-cocos2d)