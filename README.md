# Water Sort Puzzle Solver
A screenreader-solver-autoclicker for the Android game Water Sort Puzzle written in Mathematica

## The Game

[Water Sort Puzzle](https://play.google.com/store/apps/details?id=com.gma.water.sort.puzzle) is a phone game where you try to arrange differently colored liquids in test tubes such that all liquids of the same color are in the same tube. Each test tube has a capacity of **4 units of liquid**. Each **layer** is one or more units of liquid of the same color in the same tube occupying adjacent blocks. Liquid can be poured from tube A to tube B only when either the top layer of both tubes are of the same color, or tube B is empty. When pouring from A to B, the top layer of A will be transferred to the top of B, until the top layer of A is depleted or B is full. Each pour transfers integer units of liquid. Multiple non-interfering pours can happen simultaneously, which is to me the biggest fun of the game.

Many early levels of the game can be beaten with greedy algorithm. Later there are levels where some search is necessary.

To automate the game process, there are 3 functions to be implemented: **screen reading**, **level solving**, **click planning**.

## Screen Reading

Screen reading consists of ## parts: find test tubes on the screen, find the colors in each tube, translate the RGB value of the colors into category. With these steps, a screenshot of the game is converted to an array representing colors in each tube.

### Find test tubes

This function can be achieved by detecting unique features of the test tubes, namely them being surrounded by a high brightness border against a dark background, and having a specific aspect ratio. First, a simple binarization will isolate the tubes from the background. Perform a filling transform so that all pixels within the tubes are white. Detect connected regions with only white pixels, and find those with an aspect ratio of 0.726±0.020. Use these regions as masks to extract images of individual tubes.

### Find colors in tube

The geometry of a test tube is stable and robust, but sometimes there can be bubbles in the test tube interfering with color detection. Sampling a large enough area from each block and taking the median color is good enough to avoid that interference.

### Translate RGB into ordinal category

The median color found in the last step is in RGB which must be translated into categories (cyan, blue, violet, etc) labeled with ordinal numbers. As screenshots are lossless, disjoint sets with equality as union criteria should work. I used a clustering algorithm just to be safe.

## Level solving

Breadth first search is picked to be the search algorithm as it offers the shortest path to solution. Typical solutions are 20~30 levels deep and each level is roughly 1.5 nodes wide. Some optimization measures are needed to find the solution in a reasonable amount of time.

+ **Duplication rejection**. This is done by maintaining a hash table of all visited states. If a state exists in the hash table, it will not be accepted into the queue.
+ **Reject suboptimal moves**. When two tubes both hold only one type of fluid, pouring one into another and going the other way are equivalent in terms of finding a solution. Pouring more liquid takes more time in the game, so the move to pour the tube with less liquid into the other is always picked over the other way around.
+ **Deepening when not branching**. This takes advantage of the narrow shape of the search tree. Essentially only states with more than one available move will be entered into the queue. If a state has only one available move, it will be executed upon the state until the resulting state has more than one available move. This puts depth in the queue out of order, and may lead to suboptimal solution in terms of depth. But it does significantly accelerate the finding of a solution.
