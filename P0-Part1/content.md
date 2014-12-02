---
title: Build your own 2048 with SpriteBuilder and Cocos2D - Part 1!
slug: part-1
---                  

This tutorial will explain in detail how to build the popular game _2048 _from scratch, using SpriteBuilder and Cocos2D. The gameplay itself is simple but coding the game comes alongside with some puzzles and challenges. You will learn a lot in the next couple of hours!

You can find the entire code for this tutorial on [GitHub](https://github.com/MakeGamesWithUs/2048-SpriteBuilder-Tutorial). The commits follow the structure of this tutorial so you are also able to take a look at the code of the intermediate steps.

# Set up the basic structure in SpriteBuilder

A good starting point for every SpriteBuilder project is creating the outline of the game. For 2048 this will be rather simple. We are going to work with only one scene. We will add a grid to that scene and we will create a ccb file that will represent a single tile in our game.

First, **create a new SpriteBuilder project**. The first change we need to apply to the project is the device orientation. 2048 is a portrait game so let's change the game orientation to portrait:

![](https://static.makegameswith.us/gamernews_images/etwOAcHGQ2/SpriteBuilder_Portrait.png)

In the next step remove the "SpriteBuilder" label from the scene (select the label and hit backspace).

## Adding the Grid

Now we can add the grid to the screen. We will only add a grid background in SpriteBuilder. The 16 cells (4x4) that hold the game tiles will be rendered in code.

![](https://static.makegameswith.us/gamernews_images/xmxo2X0q4D/Grid.png)

For the Grid background we use a _Color Node_. Drag the _Color Node _onto the stage of MainScene.ccb. Apply the following settings to the node:

*   The anchor point should be (0.5, 0.5)
*   Position type should be _in percent of parent container_
*   The position should be (50%, 50%), this will center the node
*   The size needs to be (300,300)

Now a 300x300 grid background should be centered within _MainScene_.

We also need to set up some code connections for the grid. The grid will have a custom class - this is where the most game logic will be located and the _MainScene_ will have a variable that references the grid. Select the grid and open the code connections tab:

![](https://static.makegameswith.us/gamernews_images/AEkDSwknKi/Grid_CodeConnect.png)

## Adding Score Labels

Another important part of the _2048 _UI are labels that display the score of the current game and the highscore. We are going to add these labels to _MainScene _as well. Add four instances _Label TTF _above the grid.

![](https://static.makegameswith.us/gamernews_images/6c1Py4XW81/ScoreLabels.png)

The labels should be set up as following:

*   Top label font size: 18
*   Bottom label font size: 32
*   Position Reference Corner: Top left
*   Horizontal and Vertical Alignment: Center

Positioning the labels from the top left corner will make the interface look good on 3.5 and 4 inch phones. You can switch between previews for both device types through the _Document -&gt; Resolution_ menu in the top menu.

The labels also need code connections so that we can update the score and the highscore within our game code later on. Add code connections for both labels:

![](https://static.makegameswith.us/gamernews_images/9Dgh1QnzMs/Screen Shot 2014-04-07 at 15.05.23.png)

![](https://static.makegameswith.us/gamernews_images/L78H9WeaQX/Screen Shot 2014-04-07 at 15.05.39.png)

Name the code connection variables __scoreLabel _ and __highscoreLabel _and make sure the selected target is _Doc root var_.

## Add Tiles

We will use SpriteBuilder to create a prototype tile. We will instantiate 16 of these tiles in code, but we will define the layout of them in SpriteBuilder. Let's start by creating a new CCB file:

![](https://static.makegameswith.us/gamernews_images/Iv5tqkuw8M/Screen Shot 2014-04-07 at 15.18.34.png)

The type of the new CCB file should be _Node_.

![](https://static.makegameswith.us/gamernews_images/7331tpFMrM/Screen Shot 2014-04-07 at 15.23.54.png)

Select the root node of _Tile.ccb_ and set the size to be (70,70). This way the four tiles in each row will use 280 out of 300 points and we have 20 points left for margins between the tiles. In our version of _2048 _each tile will have a solid background color that will change whenever the value of a tile changes. Since we need to modify the behaviour of this tile in code, we need to link it to a custom class:

![](https://static.makegameswith.us/gamernews_images/ESHNOrFRQo/Screen Shot 2014-04-07 at 15.41.48.png)

Link the root node of _Tile.ccb_ to a class called _Tile._

Now we can work on adding a background color to the tile. The easiest way to apply a background color to this tile is adding a _Color Node _to _Tile.ccb. _Add a _Color Node_ by dragging it from the left panel to the timeline on the bottom and dropping it on top of the root node (CCNode):

![](https://static.makegameswith.us/gamernews_images/jApAogzjIa/Screen Shot 2014-04-07 at 15.28.33.png)

You now need to set up the color node to fill the entire root node by setting the content size type to be _in % of parent container_ and the _content size_ to (100%,100%):

![](https://static.makegameswith.us/gamernews_images/7X7VszwxVT/Screen Shot 2014-04-07 at 15.34.52.png)

You can also choose a color for the backgroud node. When we finalize the game we will change the color of the tile in code, until then the game will use the color you choose here for all tiles. We need to set up a variable that references this _Color Node_ so that we can change the color in code:

![](https://static.makegameswith.us/gamernews_images/OnflKLDk11/Screen Shot 2014-04-07 at 15.41.37.png)

Link the _Color Node_ to a variable called __backgroundNode_ and set the target to _Doc root var_. We are very close to completing the basic setup in SpriteBuilder and diving into the code.

The only step left is adding a label to the tile that will display the current value of it. Drag a _Label TTF _from the node library and add it as a child of the _Color Node_. You can either do this by dropping the label to the stage or to the timeline:

![](https://static.makegameswith.us/gamernews_images/eGHZkBq45e/Screen Shot 2014-04-07 at 15.50.12.png)

Once you have added the label you need to change a couple of settings:

![](https://static.makegameswith.us/gamernews_images/na4kUgEQWG/Screen Shot 2014-04-07 at 15.54.34.png)

*   Center the label by choosing the positioning type _in % of parent container_ and choosing (50%,50%) as position
*   Set the font size to 42
*   Check the checkbox _Adjust font size to fit._ This will automatically reduce the font size for larger numbers to make the text fit the specified dimensions
*   Set the dimensions to (70,70)

Last but not least we need a code connection for this label - we will want to change the value it displays when we merge tiles:

![](https://static.makegameswith.us/gamernews_images/xOZEauPxv2/Screen Shot 2014-04-07 at 16.15.44.png)

Name the variable __valueLabel _ and assign it to _Doc root var_.

Now we have the basic outline set up in SpriteBuilder including a grid, tiles and score labels. As a next step **publish the SpriteBuilder project **and open the Xcode project. Let's start coding!

# Setup project in Xcode

Before we start implementing the actual game logic we need to create classes and variables for the code connections we have created in SpriteBuilder. In Xcode you can create a new class by selecting _File -&gt; New File... _and selecting _Objective-C _class in the next step. We need to create one class called _Grid _and one class called _Tile._

Let's start with the _Grid_ class:

![](https://static.makegameswith.us/gamernews_images/TefLAFf4hd/Screen Shot 2014-04-07 at 16.40.36.png)

Since the Grid has a type of _Color Node_ in SpriteBuilder it needs to inherit from _CCNodeColor_. The Objective-C class always needs to match the node type in SpriteBuilder.

The second class we need to add is the _Tile _class. It needs to be subclass of _CCNode_:

![](https://static.makegameswith.us/gamernews_images/CTLY6hG5Wh/Screen Shot 2014-04-07 at 17.06.10.png)

Now that we have created both classes we need to set up the variables for the connections we defined in SpriteBuilder. Since all of these variables are private (no other class needs to see them) we will add all variable definitions to the _.m _files of our classes.

Let's start with _MainScene_. Open _MainScene.m _in Xcode. Add variables and import statements to make _MainScene.m_ contain the following code:

    #import "MainScene.h"
    #import "Grid.h"
    @implementation MainScene {
    	Grid *_grid;
    	CCLabelTTF *_scoreLabel;
    	CCLabelTTF *_highscoreLabel;
    }
    @end

The above lines import the _Grid_ class and create variables that reference the grid and both score labels that we placed in this scene using SpriteBuilder.

Next, open _Tile.m_ and add the following variables:

    @implementation Tile {
    	CCLabelTTF *_valueLabel;
    	CCNodeColor *_backgroundNode;
    }

Now we have set up all required code connections and you should be able to run the game for the first time. Hit the run button in Xcode and you should see this game on the iPhone simulator:

![](https://static.makegameswith.us/gamernews_images/cfeOwooHax/iOS Simulator Screen shot 07 Apr 2014 17.21.25.png)

Pretty empty - but it's up and running. Next we will render 16 cells as background for our grid.

# Render a grid background

We are going to implement the background rendering in the grid class. Open _Grid.m_ in Xcode.

For _2048 _we need a 4x4 grid with 16 tiles spread out with a constant margin. Since we are good developers we like to create programs that have a certain flexibility - this means we don't want our program to break when simple parameters change. An example: if we come up with a good solution it should be fairly easy to change the game and make it use a 5x5 grid instead of a 4x4 one.

This means we will have to calculate the position of each cell in code, instead of defining them initially upfront. The relevant factors for positioning the 16 cells are:

*   the size of the grid
*   the size of the tiles
*   the margin between the tiles

We are going to implement a mechanism that reads the grid size and the tile size and calculates a margin automatically. We will need to add variables and constants to our _Grid_ class to store all this information:

    @implementation Grid {
    	CGFloat _columnWidth;
    	CGFloat _columnHeight;
    	CGFloat _tileMarginVertical;
    	CGFloat _tileMarginHorizontal;
    }
    static const NSInteger GRID_SIZE = 4;

We create 4 float variables that store information about the grid and one constant that defines the amount of tiles in the grid - by default we assume a 4x4 grid.

Now we need to add a method that renders 16 empty cells to our grid. We will call it _setupBackground:_

    - (void)setupBackground
    {
    	// load one tile to read the dimensions
    	CCNode *tile = [CCBReader load:@"Tile"];
    	_columnWidth = tile.contentSize.width;
    	_columnHeight = tile.contentSize.height;
            // this hotfix is needed because of issue #638 in Cocos2D 3.1 / SB 1.1 (https://github.com/spritebuilder/SpriteBuilder/issues/638)
            [tile performSelector:@selector(cleanup)];
    	// calculate the margin by subtracting the tile sizes from the grid size
    	_tileMarginHorizontal = (self.contentSize.width - (GRID_SIZE * _columnWidth)) / (GRID_SIZE+1);
    	_tileMarginVertical = (self.contentSize.height - (GRID_SIZE * _columnWidth)) / (GRID_SIZE+1);
    	// set up initial x and y positions
    	float x = _tileMarginHorizontal;
    	float y = _tileMarginVertical;
    	for (int i = 0; i < GRID_SIZE; i++) {
    		// iterate through each row
    		x = _tileMarginHorizontal;
    		for (int j = 0; j < GRID_SIZE; j++) {
    			//  iterate through each column in the current row
    			CCNodeColor *backgroundTile = [CCNodeColor nodeWithColor:[CCColor grayColor]];
    			backgroundTile.contentSize = CGSizeMake(_columnWidth, _columnHeight);
    			backgroundTile.position = ccp(x, y);
    			[self addChild:backgroundTile];
    			x+= _columnWidth + _tileMarginHorizontal;
    		}
    		y += _columnHeight + _tileMarginVertical;
    	}
    }

This is a lot of code, but no worries, all of it is fairly straightforward. First we load a _Tile.ccb_ to read the height and width of a single tile. Then we subtract the width of all tiles we need to render from the width of the grid to calculate the available width. Once we have the available width we can calculate the available horizontal margin between tiles. We do the same for the height and the vertical margin.

Once we know the margins we run through a two dimensional loop to create all tiles. We start at the first row (bottom) and render all columns of the first row (from left to right). Once we reach the last column we move to the next row. We repeat until we reach the last column of the last row (top right). The following image visualizes the loop that renders the tiles:

![](https://static.makegameswith.us/gamernews_images/v8G0TMWMrG/RenderingGrid.png)

Now that you understand the rendering code we just need to call it. When working with scenes created in SpriteBuilder the method _didLoadFromCCB_ is the right place to perform modifications that shall happen as soon as the scene gets initialized.

Let's call our new method from _didLoadFromCCB_ by adding this implementation to _Grid.m_:

    - (void)didLoadFromCCB {
    	[self setupBackground];
    }

Now the background will be rendered as soon as the _MainScene.ccb _is loaded. You can run the app now and should see following result on the screen:

![](https://static.makegameswith.us/gamernews_images/zeUMJ8VlYV/iOS Simulator Screen shot 07 Apr 2014 18.19.47.png)

Well done! This is starting to look like a real game. In the next step we are going to spawn our first tiles.

# Spawn the first tiles

In this step we are going to make a lot of progress. We will create a data structure for our grid (a 2D array) and we will add methods that will spawn tiles and add them to our data structure and to our visual grid. This chapter will also be a lesson about breaking a large problem down into many small problems. Whenever we need to write a complex piece of code breaking down the problem into smaller ones should be the first step.

**The large problem: **We need to spawn a certain amount of randomly positioned tiles when our program starts. We need to add the tiles to a data structure and we need to add them visually to the grid.

**Small problems:**

*   We need the capability to spawn a random tile
*   We need to spawn **n **random tiles when the program starts
*   We need a data structure to store spawned tiles
*   We need to determine where (visually) on the grid a tile needs to be added

The next step is transforming these small problems into methods that we can implement in our program.

**Methods to solve small problems:**

*   **addTileAtColumn:Row: **adds a tile to the data structure and adds it to the visual grid with an animation
*   **spawnRandomTile: **determines a random position and uses the _addTileAtColumn:Row: _method to add a tile at that position
*   **spawnStartTiles: **calls _spawnRandomTile _**n **times
*   **positionForColumn:Row: **a utility method that is used by _addTileAtColumn:Row: _to determine where a tile needs to be added to the visual grid

As you can see, all of these methods are not too complicated - the most complicated step is breaking the big problem down into smaller problems. Before we can start implementing these methods we need to add an import statement, a constant and two variables. Add this import statement to the top of _Grid.m_:

    #import "Tile.h"

	Now we can use the _Tile_ class in our methods. Additionally add these two member variables:

    @implementation Grid {
        ..
    	NSMutableArray *_gridArray;
    	NSNull *_noTile;
    }

The __gridArray_ is a two dimensional array that will store the tile for each index of the grid. The __noTile_ variable will represent an empty cell in the __gridArray_. Because arrays in Objective-C can only store objects (not primitive types or _nil_) we need to use an instance of _NSNull_ to represent an empty slot in the grid.

We also need to add a constant that will store how many start tiles we want to spawn. Add this constant below the already existing _GRID_SIZE _constant:

    static const NSInteger START_TILES = 2;

Now we can start implementing the different methods and putting the parts together!

### Determining the position for a new tile

First we are going to add the _positionForColumn:Row: _method. This method uses the information we stored about the grid (column sizes, margins) to calculate a point for a given tile index. The implementation are only a few lines. Add these lines to _Grid.m_.

    - (CGPoint)positionForColumn:(NSInteger)column row:(NSInteger)row {
    	NSInteger x = _tileMarginHorizontal + column * (_tileMarginHorizontal + _columnWidth);
    	NSInteger y = _tileMarginVertical + row * (_tileMarginVertical + _columnHeight);
    	return CGPointMake(x,y);
    }

We will use this method momentarily when adding tiles to the game.

### Add a tile at a certain row and column

The next method we are going to implement is the one that adds a tile at a specified row and column. Add this method to _Grid.m_:

    - (void)addTileAtColumn:(NSInteger)column row:(NSInteger)row {
    	Tile *tile = (Tile*) [CCBReader load:@"Tile"];
    	_gridArray[column][row] = tile;
    	tile.scale = 0.f;
    	[self addChild:tile];
    	tile.position = [self positionForColumn:column row:row];
    	CCActionDelay *delay = [CCActionDelay actionWithDuration:0.3f];
    	CCActionScaleTo *scaleUp = [CCActionScaleTo actionWithDuration:0.2f scale:1.f];
    	CCActionSequence *sequence = [CCActionSequence actionWithArray:@[delay, scaleUp]];
    	[tile runAction:sequence];
    }

This method performs a couple of tasks. First we load the tile by loading the CCB file and storing it in a local variable. Then we also store this tile in the grid array. We set the scale of the tile to 0 because we want the tile to appear with a scale up animation. Then we add the child to the grid. We define the position of the tile using the _positionForColumn:row: _method. Then we create a little action sequence that  forms a spawn animation. The tile starts with a scale of 0 and is invisible. We define a action sequence that waits for 0.3 seconds and then scales the tile up to it's full size in 0.2 seconds.

That's it! Now he have a method to add a tile at any position in the game. Now there's not much more code to go and we will be spawning random tiles.

### Spawning a random tile

The next method that we are going to add will determine a random free position on the grid to spawn a new tile. The easiest way to do this is having a loop that continues generating a random tile index until it finds a free position on the grid. _Note: this is not the most efficient way to do this, once there are only a few spots left on the grid the program will generate many random positions that will already be occupied by other tiles. However, this approach is absolutely fine for this type of game. _Add the _spawnRandomTile _method to _Grid.m_:

    - (void)spawnRandomTile {
    	BOOL spawned = FALSE;
    	while (!spawned) {
    		NSInteger randomRow = arc4random() % GRID_SIZE;
    		NSInteger randomColumn = arc4random() % GRID_SIZE;
    		BOOL positionFree = (_gridArray[randomColumn][randomRow] == _noTile);
    		if (positionFree) {
    			[self addTileAtColumn:randomColumn row:randomRow];
    			spawned = TRUE;
    		}
    	}
    }

This method picks a random position and checks if it is occupied. We test occupation by checking if the tile for the index is a __noTile_ (these represent empty slots). If the position is occupied the loop continues and generates a new random number, if the picked position is free the loop terminates and the method uses the _addTileAtColumn: _to add a tile at that position. Now all we need to do is call this method multiple times and we will be able to spawn our start tiles!

### Spawn multiple start tiles

Now we are going to call the _spawnRandomTile _method for each start tile. Add this method to _Grid.m_:

    - (void)spawnStartTiles {
    	for (int i = 0; i < START_TILES; i++) {
    		[self spawnRandomTile];
    	}
    }

	Very straightforward! One last change and we can finally run the game and watch the tiles spawn.  Change your _didLoadFromCCB_ method to look like this:

    - (void)didLoadFromCCB {
    	[self setupBackground];
    	_noTile = [NSNull null];
    	_gridArray = [NSMutableArray array];
    	for (int i = 0; i < GRID_SIZE; i++) {
    		_gridArray[i] = [NSMutableArray array];
    		for (int j = 0; j < GRID_SIZE; j++) {
    			_gridArray[i][j] = _noTile;
    		}
    	}
    	[self spawnStartTiles];
    }

First we initialize our __noTile_ variable - you remember we use this variable to represent an empty slot in the grid. The _null_ method of _NSNull_ always returns the same instance and we will use this instance to check if slots are free or not.

In the second step we initialize the __gridArray_ and store the __noTile_ value for each index. Because arrays in Objective-C don't allow to store nil values we need to set up our grid with these initial values that represent empty slots.

These were a lot of steps! But we haven't only built the functionality to spawn start tiles, we have implemented many methods that we will be reusing moving forward.

**Now it is time to run the app and check if everything worked out. **When the app started you should see something similar to this:

![](https://static.makegameswith.us/gamernews_images/qsIhlwSzKr/iOS Simulator Screen shot 08 Apr 2014 22.37.05.png)

Two spawned tiles! Well done! We added a lot of code in this step. In case something is not working as expected you should compare your results to the [solution on GitHub](https://github.com/MakeGamesWithUs/2048-SpriteBuilder-Tutorial/blob/6ab53171b8e2e7c8f46004e5a9be9eeda4e1bc39/Source/Grid.m).

We set up the basic project in this step. We added the data model for our game and already included a system that allows to add tiles with animations. **Well done!**

[You can now move on to part 2 of the tutorial where we will add user interaction and tile movement to this game!](https://www.makegameswith.us/gamernews/387/build-your-own-2048-with-spritebuilder-and-cocos2d)
