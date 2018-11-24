# Snake #
This tutorial will walk us through making Snake. A completed version of the game can be played [here](https://peach-justice.glitch.me).

In Snake, the player controls the snake to collect as much food as possible, and avoid running into anything else in the game. This tutorial will follow the same format as the Pong tutorial, which means we'll look at the following.

1. The gameplay.  
A very basic description of how Snake works.  

2. What the framework needs.  
The information we'll need to pass to the framework to run the game. We'll list out the canvas attributes, actions, and different sprites needed for the game. We'll also learn about some new features we can use in the framework: timers and scaling.

3. Sprites' initial values. 
Details about sprites at the start of the game. We'll list out the different attributes and components for each sprite, and their initial values. We'll also learn how we can use the `start` function for intializing, or setting up starting values, for different attributes of the sprites. 

4. Sprites' dynamic values. 
Details about sprites that can change as we play the game. We'll look at values that need to be changed to make our sprites move or react to collisions with other objects in the game. With this game, we'll learn how we can add sprites to the game after it's started, and how to remove them as well.

## 1. the gameplay ##
The game has a single snake, pieces of food, and four walls that create a border. The snake is controlled by a single player, and can be moved in any direction. At the start of the game, the snake is very short and moves slowly towards a piece of food to the right. When the player moves the snake to eat food, the snake grows in length and moves a little faster. As food is eaten, a new piece of food appears in a random location. The player continues moving the snake to eat as much food as possible. The game ends when the snake runs into a wall, or itself.

## 2. what the framework needs ##
To start, here's some _skeleton_ code for our game. Calling it _skeleton_ code just means that it's not incomplete, and there's a lot of code that we'll need to fill in.

_skeleton code for snake_
```javascript
//variables for framework
var canvas = {};
var input = {};
var timers = {};
var sprites;

//declare sprites
var snake = {};
var food = {};
var topWall = {};
var bottomWall = {};
var leftWall = {};
var rightWall = {};

//function to easily create snake pieces
var SnakePiece = function(){

};

//code to finish setting up sprites should go here
//code to setup food
//code to setup top wall
//code to setup bottom wall
//code to setup left wall
//code to setup rightwall
//code to setup snake

//list of initial sprites for the game
var sprites = [snake, topWall, bottomWall, leftWall, rightWall, food];

//setup object for framework
var myGame = {};
myGame.canvas = canvas;
myGame.sprites = sprites;
myGame.input = input;
myGame.timers = timers;

//setup and start game
Game.setup(myGame);
Game.start();
```
For our game, there are four major items we'll need to provide to the framework:

 1. `canvas`
 2. `input`
 3. `sprites`
 4. `timers`

 ### `canvas` ###

When we created Pong, the canvas had a width of `640pixels` and a height of `640pixels`. Notice that the units are in pixels. Part of the challenge in working with pixel as units is calculating exactly where to position things. A great example is positioning the ball. To center the ball, we had to calculate the midpoint for the width and height of the screen. Another good example is setting up the bottom wall. That requires setting the y postion based off of the height of the canvas and the height of the wall. 

To get around this for snake, we'll use an attribtue in the canvas called `scaleFactor`. Before describing how this helps us with Snake, let's look at an example. In snake, it just so happens that the food has the dimensions as the ball from Pong: a width of `32pixels` and a height of `32pixels.` But what if we wanted the food to have a width of 1 and a height of 1? We could divide the width and the height by `32pixels`. And then the widht and height would be `1unit` by `1unit`. Notice the change in units, `pixesl` to `unit`.

In Snake, we really want everything to move like it's in a grid. When the snake moves, it always moves one full square. To accomplish this, we'll set `scaleFactor` to 32. For us, this means `1unit` of movement is equivalent to moving `32pixels`. While using `scaleFactor` we'll also set the width and the height to 20. This means our grid for the game is `20units` by `20units`. If we wanted to know how big the game is in pixels, we would just solve `scaleFactor * width` and `scaleFactor * height` which is 640 and 640.

Here's a table for all of the values.

| attribute                 | value            |
|--------------------------:|-----------------:|
| `canvas.width`            | 20               |
| `canvas.height`           | 20               |
| `canvas.scaleFactor`      | 32               |
| `canvas.id`               | `"canvas"`       |
| `canvas.background.color` | `"rgb(0, 0, 0)"` |

_code for setting up canvas_
```javascript
canvas.background = {};
canvas.width = 20;
canvas.height = 20;
canvas.scaleFactor = 32;
canvas.id = "canvas";
canvas.background.color = "rgb(0, 0 0)";
```

### `input` ###

For the input, we just need four keys so we can move the snake up, down, left, and right. We'll use WASD for this. And since there's only one player, we'll use simple names for the actions like up, down, left, and right.

| action name   | key code |
|--------------:|---------:|
| `input.up`    | `"KeyW"` |
| `input.left`  | `"KeyA"` |
| `input.down`  | `"KeyS"` |
| `input.right` | `"KeyD"` |

_code for setting up input_
```javascript
input.up = "KeyW";
input.left = "KeyA";
input.down = "KeyS";
input.right = "KeyD";
```
 
### `sprites` ###
We learned that players move the snake to eat food, and try to avoid running into walls or the snake itself. This means we'll need four sprites for the walls, and a fifth sprite for the food.

Now is a good time to try and breakdown the snake. The snake is made up of multiple pieces, with each piece moving based off of the piece in front of it. This means we need something to control how all the pieces move, and we can use a sprite for that. Also, the head of the snake is the most important piece. The head is what eats food, runs into walls, and runs into other pieces of the snake. This means we'll also need a sprite for the head of the snake. That's two more sprites, brining us to a total of 7.

1. the top wall
2. the bottom wall
3. the left wall
4. the right wall
5. the food
6. the snake
7. the snake's head

We'll explore this more in mroe detail in the next few sections, but the snake is really different from the head and other pieces of the snake. We just use it to _manage_ all the pieces of the snake, like moving them around and changing the direction the snake travels in. Other than that, it's not all that interesting and we'll see why.

One more important note. It's important to remember that this is just **one** particular way to create the snake. This particular way is nice because we can learn many new concepts that are helpful for making games.

### `timers` ###
If we watch gameplay of Snake, we may notice how the snake doesn't move all the time. There's a delay between movements: it moves, then pauses. After a short pause, it moves and pauses again. As the snake eats food, the snake moves faster by having shorter pauses.

We can make the snake move like this by using timers. To setup a timer, we set how long the timer should run before going off. We can also check if a timer has gone off, which we'll see later. To setup a timer, we just need to provide a name, and the amount of time in milliseconds.

For our game, we'll name our timer `move`, since we're using it to pause and start movement. We'll set it to be 500milliseconds.

| name        | duration |
|------------:|---------:|
|`timers.move`|500       |

_code for setting up timers_
```javascript
timers.move = 500;
```

## 3. sprites' initial values ##
For Snake, here is a list of *all* the different attributes and components we'll use, and basic descriptions of each.
 - attributes
   - *name*: name of the sprite
   - *width*: the width of the sprite, in units
   - *height*: the height of the sprite, in units
   - *isSensor*: a special attribute that affects physics
 - components
   - *tags*: keywords for the sprite
   - *position*: the x and y coordinate, in units, for where the sprite is positioned 
   - *image*: the image to use for the sprite
   - *hitBox*: the dimensions of the box used for detecting collisions
   - *physics*: allows the sprite to have a x and y velocity, in units per frame
   - *start*: a function which can change a sprite's values right before the game starts
   - *update*: a function which can change a sprite's values each frame
   - *onCollision*: a function to:  
     1. check what a sprite has collided with and 
     2. change the sprite's values based off of the collision

This table shows the values that different sprites will have for attributes and components. A dash, `-`, means that the sprite doesn't need that attribute or component.

For Snake, there's a new attribute and component we'll use.

  - `isSensor`: This is is an attribute that affects the physics of a sprite. If this is true, we still get notified when a collision occurs, but the physics will allow sprites to collide. This is really helpful for the interactions between the head of the snake and food. Otherwise, the head of the snake would move backwards after it collides with food. 

  - `tags`: is a list of keywords that we can assign to a sprite. Then later, we can search for sprites using a keyword, and get back a list of sprites which have the keyword.

|attribute      | left wall | right wall | top wall  | bottom wall | food        | snake     | head of snake        | piece of snake      |
|:--------------|----------:|-----------:|----------:|------------:|------------:|----------:|---------------------:|--------------------:|
|`name`         |-          |-           | -         |-            | `"food"`    | `"snake"` | -                    |  -                  |
|`width`        | 1         |1           | 20        |20           | 1           | -         | 1                    | 1                   |
|`height`       | 20        |20          | 1         |1            | 1           | -         | 1                    | 1                   |
|`isSensor`     | `true`    |`true`      | `true`    |`true`       | `true`      | -         | `true`               | `true`              |

| component     | left wall | right wall | top wall  | bottom wall | food        | snake     | head of snake        | piece of snake      |
|:--------------|----------:|-----------:|----------:|------------:|------------:|----------:|---------------------:|--------------------:|
|`tags`         |`["wall"]` | `["wall"]` | `["wall"]`|`["wall"]`   | -           | -         |-                     |`["snake piece"]`    |
|`image.src`    |-          |-           |-          |-            |[food][image]|-          |[head of snake][image]|[snake piece][image] |
|`position.x`   | -1        | 20         | 0         | 0           | 13          | -         | 10                   | -                   |
|`position.y`   | 0         | 0          | -1        | 20          | 10          | -         | 10                   | -                   |
|`hitBox.width` |1          | 1          | 20        | 20          | 1           | -         | 1                    | 1                   |          
|`hitBox.height`|20         | 20         | 1         | 1           | 1           | -         | 1                    | 1                   |
|`physics`      |-          |-           |-          |-            | -           | -         | `true`               | `true`              |
|`start`        |`no`       |`no`        |`no`       |`no`         |`no`         |`yes`      |`no`                  |`no`                 |
|`update`       |`no`       |`no`        |`no`       |`no`         |`no`         |`yes`      |`no`                  |`no`                 |
|`onCollision`  |`no`       |`no`        |`no`       |`no`         |`no`         |`no`       |`yes`                 |`no`                 |       

[image]: https://github.com/pyvelepor/framework/raw/snake/sprites/pong/ball.png

You may have noticed several things.

 1. The values for the position and hit box are smaller, which is because of the scale factor that we set on the canvas. We just need to remember that `1unit` is `32pixels`.
 2. We're using `tags` for the walls and snake pieces. All the walls and the all the snake pieces are almost the same, but we need a way of checking if we're colliding with any of them. By assining tags, we can check to see if the sprite has the tag `wall` or `snakePiece` when `onCollision` gets called. You might be wondering why we don't just use `name` instead. It's because multiple spirtes **cannot** have the same name.     
3. `snake` has `start` and `update` functions, but not a `onCollision` function. Remember that `snake` is really just to manage all the pieces of the snake, like making them change direction and move. Since it's not a sprite that anything in the game interacts with, it doesn't need a `onCollision` function. 
4. Only the head of the snake has an `onCollision` function. Since the head is the only piece of the snake that can run into other sprites, it's the only piece that needs an `onCollision` function.
5. Other pieces of the snake don't have any functions. Two reasons: (1) the head of the snake checks for collisions and (2) `snake` manages everything else.

Here's the code for one of the left wall. The code can be copied and changed for the other the right, top and bottom walls.

_code for left wall_
```javascript
//setup sprite for top wall
leftWall.position = {};
leftWall.hitBox = {};
leftWall.tags = ["wall"];
leftWall.width = 1;
leftWall.height = 20;
leftWall.position.x = -1;
leftWall.position.y = 0;
leftWall.hitBox.width = 1;
leftWall.hitBox.height = 20;
```

And here's the code for setting up the food sprite.

_code for food_
```javascript
//setup food sprite
food = {};
food.position = {};
food.image = {}
food.hitBox = {};
food.name = "food";
food.width = 1;
food.height = 1;
food.isSensor = true;
food.image.src = "https://github.com/pyvelepor/framework/raw/snake/sprites/pong/ball.png";
food.position.x = 13;
food.position.y = 10;
food.hitBox.width = 1;
food.hitBox.height = 1;
```

And skeleton code for the snake

_skeleton code for the snake sprite_
```javascript
snake.name = "snake";
snake.start = function(){
  //code for starting the snake goes here
};

snake.update = function(){
  //code for updating the snake goes here
};
```
## 4. sprites' dynamic values ##
In our version of Snake, `snake` and the head have most of the dyanmic values and change because of our timer (`move`), actions from keyboard input, and collisions. So this section will only focus on those two sprites, and we can breakdown our dynamic values by looking at what happens when:

1. the game starts
2. the timer goes off
3. an action occurs
4. a collision occurs

### 1. the game starts ###
When the game starts, we need to make sure that the snake has only one piece: the head. First, to make it easier to create pices of the snake, we can make a function that creates a sprite with all of the default values for a piece of the snake.

_function to create snake pieces_
```javascript
function SnakePiece(){
  this.position = {};
  this.image = {};
  this.hitBox = {};
  this.tags = ["snake piece"];
  this.oldPosition = {};
  this.isSensor = true;
  this.physics = true;
  this.width = 1;
  this.height = 1;
  this.position.x = 1;
  this.position.y = 1;
  this.image.src = "";
  this.hitBox.width = 1;
  this.hitBox.height = 1;
  this.oldPosition.x = 1;
  this.oldPosition.y = 1;
}
```

With this function, we're ready to start adding code to the `start` function of the snake. To start, we'll add another attribute to snake called `direction`, which will let us update which direction the snake moves. It's an object that has an `x` and `y` property, like `position` and `velocity`. The default direction causes the snake to move right, so the x values will be 1 and the y value will be 0.

_initial direction_
```javascript
//create JavaScript object for direction
snake.direction = {};

//create x and y properties for direction
snake.direction.x = 1;
snake.direction.y = 0;
```

Next, we need to add an _array_ named `pieces` to `snake`, which we'll use for tracking and managing all of the snake pieces. When we do, we also want to make sure that it has a single snake piece.

_setting up array for snake pieces_
```javascript
//create an array named pieces
snake.pieces = [];

//add a single snake piece for the head
snake.pieces.push(new SnakePiece());
```
Next, we need to set the x and y position of the head to the correct initial values: 10 and 10.

_set head's position to correct values_
```javascript
snake.pieces[0].position.x = 10;
snake.pieces[0].position.y = 10;
```

Next, because `SnakePiece` doesn't add `onCollision` to the pieces, we need to add this function to the head. For now, we'll add a function that does nothing, and we'll update the code when we look at changing values because of collisions.

_add empty `onCollision` function to head_
```javascript
//add empty `onCollision` function to head of snake
//`snake.pieces[0]` refers to head of snake
snake.pieces[0].onCollision = function(sprite){};
```

In Pong, all of our sprites existed at the beggining of the game, so we added them all to the game before we started it. But when we start snake, all of our sprites, including the head, haven't been created yet. It's important to remember that after we've created a sprite, we make sure we ***add the head to the game***.  If we forget to add this or other sprites, the game doesn't know about them. To to this, we need to use `Game.sprites.add` to add sprites that are created after the game has been started.

_add head to game_
```javascript
//add the head to the game
//`snake.pieces[0]` refers to head of snake
Game.sprites.add(snake.pieces[0]);
```

### 2. the timer goes off ###
We make the snake move by using a timer. Whenver the timer goess off, we know it's time to move the snake. When can check if our timer has gone off by using `Game.timers(<name>).ready()` where `<name>` should be then name of the timer we created. In our version of Snake, we named the timer `move`. This returns `true` if the timer has gone off and `false` if it hasn't. We can use this as the condition of an if statement, so if the timer has gone off, we can move all the snake peices.

_check if timer has gone off_
```javascript
if(Game.timers.withName("move").ready()){
  //code to move snake pieces
}
```

If the timer has gone off, here's how we make the snake move:

1. We start by moving the **zeroth** piece of the snake, the head.
  - We save the head's current position by setting `position.x` and `position.y` equal to `oldPosition.x` and `oldPosition.y`.
  - We updated the head's current position using `snake.direction`. 
2. We then move the **first** piece of the snake, the next piece in `snake.pieces`.
  - We save the **first** piece's current position by setting `position.x` and `position.y` equal to `oldPosition.x` and `oldPosition.y`.
  - We update the **first** pieces current position by setting it to **zeroth** pieces old position.
3. We then move the **second** piece of the snake, the next piece in `snake.pieces`.
  - We save the **second** piece's current position by setting `position.x` and `position.y` equal to `oldPosition.x` and `oldPosition.y`.
  - We update the **second** pieces current position by setting it to **first** pieces old position.

You may have noticed a few patterns:

1. We update each piece of the snake in order. We start with zero, then one, then two, and so on.
2. We save the piece's old position first. This is really important since other pieces of the snake need this to move correctly.
3. We update the position of a snake piece last, using the `oldPosition` of the piece in front it. For example, if we're trying to move the **third** piece of the snake, we use `oldPosition` from the **second** piece of the snake. If we're trying to move the **fourth** piece of the snake, we use `oldPosition` from the **third** piece of the snake.

To move all of the pieces of the snake, we'll use a while loop and a variable that let's keep track of which piece we're updating. We'll name the variable `index`. A while loop let's us repeat a bunch of code while a condition is true. In our case, the condition is whether `index` is less than the total number of snake pieces, `snake.pieces.length`. On each pass of the loop, we'll increase the value of index by 1. If we don't, the condition `index < snake.pieces.length` would always be true, and the loop would never stop.

The last thing we do, outside the loop, is reset the timer using `Game.timers.withName("move").start()`. If we don't reset the timer, the snake will stop moving.

_making the snake move_
```javascript
//variable to keep track of which snake piece is being updated
var index = 0;

//check if the timer has gone off
if(Game.timers.withName("move").ready()){

  //loop over all of the snake pieces
  while(index < snake.pieces.length){

    //1. move the zeroth piece first
    //check if `index` is the head
    if(index === 0){
      //save old position
      snake.pieces[index].oldPosition.x = snake.pieces[index].position.x;
      snake.pieces[index].oldPosition.y = snake.pieces[index].position.y;

      //update head's position using `snake.direction`
      snake.pieces[index].position.x += snake.direction.x;
      snake.pieces[index].position.y += snake.direction.y;
    }

    //2. move first through last snake pieces
    //default is that `index` is any other snake piece
    else{
      //save old position
      snake.pieces[index].oldPosition.x = snake.pieces[index].position.x;
      snake.pieces[index].oldPosition.y = snake.pieces[index].position.y;

      //update piece's position using old position of piece in front
      //`index` is current piece
      //`index - 1` is piece in front
      snake.pieces[index].position.x = snake.pieces[index - 1].oldPosition.x;
      snake.pieces[index].position.y = snake.pieces[index - 1].oldPosition.y;
    }

    //increase index by 1
    index += 1;
  }

  //reset timer
  Game.timers.withName("move").start();
}
```

### 3. an action occurs ###
For Snake, we use actions to change the direction the snake will move in. The code for the actions is very similar to the code for moving the paddles in Pong. The major difference is that instead of updating a velocity, we update `this.direction`. Here's a table that lists the four different directions, and the corresponding x and y values.

|action name                     | `this.direction.x` | `this.direction.y` |
|:-------------------------------|-------------------:|-------------------:|
|`Game.actions.withName("up")`   | 0                  | -1                 |
|`Game.actions.withName("left")` | -1                 | 0                  |
|`Game.actions.withName("down")` | 0                  | 1                  |
|`Game.actions.withName("right")`|1                   | 0                  |

And here's the code, using the values from the table.
```javascript
if(Game.actions.withName("up")){
  this.direction.x = 0;
  this.direction.y = -1;
}

else if(Game.actions.withName("left")){
  this.direction.x = -1;
  this.direction.y = 0;
}

else if(Game.actions.withName("down")){
  this.direction.x = 0;
  this.direction.y = 1;
}

else if(Game.actions.withName("right")){
  this.direction.x = 1;
  this.direction.y = 0;
}
```

### 4. a collision occurs ###
Earlier, we added an empty `onCollision` function to the head of the snake. Now, we'll add code to the function to complete our Snake game. Remember that there are three things the head of the snake can collide with walls, other pieces of the snake, and food.

When the head collides with a wall, or another piece of the snake, we need to reset the game. To do this, we need to

1. get the snake sprite
2. check if head collided with a wall
3. remove all snake pieces, except for the head, from the game
4. reset the head of the snake to center
5. reset the food to be just to the right of the snake

#### 1. get the snake sprite ####
To get the snake sprite, we can use `Game.sprites.withName(<name>)` where `<name>` is the name of the sprite: `"snake"`.

_using `Game.sprites.withName` to get the snake sprite_
```javascript
var snake = Game.sprites.withName("snake");
```

#### 2. check if head collided with a wall ####
To check if the head of the snake collided with a wall, we can see if `sprite` has a tag called `"wall"`. To check, we can use `sprite.tags.includes(<tag>)`, where `<tag>` is a string. In our case, the string is `wall`. And since we only want to do more things if the sprite is a wall, we'll use an if statement too.

_checking if `sprite` is a wall_
```javascript
if(sprite.tags.includes("wall")){
  //code to remove snake pieces goes here
}
```

#### 3. remove all snake pieces, except for the head ####
To remove all the snake pieces, we can use a while loop again and a variable called `index`, just how we did to make all the pieces move. But instead of starting `index` at 0, we start it at 1. Remember that the head is when `index` is 0, and we don't want to remove the head.

_looping over snake pieces using a while loop_
```javascript
var index = 1;
while(index < snake.pieces.length){
  //other code would go here
  ...

  //increase index to work with next snake piece
  index = index + 1;
}
```

Inside the loop before increasing the index, we can use `Game.sprites.remove(<sprite>)` to remove snake pieces from the game. This is the opposite of using `Game.sprites.add` to add sprites to the game. `<sprite>` should be the sprite that we want to remove, which will be `snake.pieces[index]`.

_removing snake pieces from the game_
```javascript
//start at 1 so we don't remove the head
var index = 1;

//loop over the snake pieces 
while(index < snake.pieces.length){
  
  //remove each piece from the game
  Game.sprites.remove(snake.pieces[index]);

  //increase index so we can remove the next snake piece
  index += 1;
}
```

After removing the snake pieces from the game, we also need to remove them from the `snake.pieces`. It's a three step process. First, we save the head by assigning `snake.pieces[0]` to a variable called `head`. Next, we empty the array by assining an empty array (`[]`) to `snake.pieces`. Last, we put the head back into the array using `snake.pieces.push`.

_removing snake pieces from `snake.pieces`_
```javascript
//save the head to a variable
var head = snake.pieces[0];

//empty the array of snake pieces
snake.pieces = [];

//put the head back into the array
snake.pieces.push(head)
```

Here's what all the code looks like put together.

_removing snake pieces and resetting `snake.pieces`_
```javascript
if(sprite.tags.includes("wall")){
  //get the snake sprite
  var snake = Game.sprites.withName("snake");

  //start at 1 so we don't remove the head
  var index = 1;

  //loop over the snake pieces 
  while(index < snake.pieces.length){
    
    //remove each piece from the game
    Game.sprites.remove(snake.pieces[index]);

    //increase index so we can remove the next snake piece
    index += 1;
  }
  //save the head to a variable
  var head = snake.pieces[0];

  //empty the array of snake pieces
  snake.pieces = [];

  //put the head back into the array
  snake.pieces.push(head)
}
```

#### 4. reset the head of the snake to center ####
To recenter the head of the snake, we can set the x and y position back to their initial values: 10 and 10.

_resetting snake's head to initial position_
```javascript
//recenter the head of the snake
head.position.x = 10;
head.position.y = 10;
```

#### 5. reset the food to be just to the right of the snake ####
Recentering the food is a two step process. First, we use `Game.sprites.withName(<name>)` to get the food, where name is `"food"`. Next, we set the x and y position back to their intitial values: 13 and 10.

_resetting food to initial position_
```javascript
//get the food sprite
var food = Game.sprites.withName("food");

//reset food back to initial position
food.position.x = 13;
food.position.y = 10;
```

Here's what the completed of the code looks like.

_removing snake pieces if head collided with wall_
```javascript
//1. check if we're colliding with a wall
if(sprite.tags.includes("wall")){
  //2. get the snake sprite
  var snake = Game.sprites.withName("snake");

  //3. remove snake pieces
  //start at 1 so we don't remove the head
  var index = 1;

  //loop over the snake pieces 
  while(index < snake.pieces.length){
    
    //remove each piece from the game
    Game.sprites.remove(snake.pieces[index]);

    //increase index so we can remove the next snake piece
    index += 1;
  }

  //save the head to a variable
  var head = snake.pieces[0];

  //empty the array of snake pieces
  snake.pieces = [];

  //put the head back into the array
  snake.pieces.push(head);

  //4. recenter the head of the snake
  head.position.x = 10;
  head.position.y = 10;

  //5. reset food
  //get the food sprite
  var food = Game.sprites.withName("food");

  //reset food back to initial position
  food.position.x = 13;
  food.position.y = 10; 
}
```

The good news is that when we collide with another piece of the snake, the code is exactly the same. The only change is checking whether we've collided with a snake piece. Instead of checking `"wall` is one of the tags, we check to see if `"snake piece"` is one of the tags.

_removing snake pieces if head collided with another snake piece_
```javascript
//1. check if we're colliding with a wall
if(sprite.tags.includes("snake piece")){
  //2. get the snake sprite
  var snake = Game.sprites.withName("snake");

  //3. remove snake pieces
  //start at 1 so we don't remove the head
  var index = 1;

  //loop over the snake pieces 
  while(index < snake.pieces.length){
    
    //remove each piece from the game
    Game.sprites.remove(snake.pieces[index]);

    //increase index so we can remove the next snake piece
    index += 1;
  }

  //save the head to a variable
  var head = snake.pieces[0];

  //empty the array of snake pieces
  snake.pieces = [];

  //put the head back into the array
  snake.pieces.push(head);

  //4. recenter the head of the snake
  head.position.x = 10;
  head.position.y = 10;

  //5. reset food
  //get the food sprite
  var food = Game.sprites.withName("food");

  //reset food back to initial position
  food.position.x = 13;
  food.position.y = 10; 
}
```

Colliding with food requiers a different set of steps.

1. check if head collided with food
2. get the snake sprite
3. add a snake piece to the game
4. move food to random location

#### 1. check if head collided with food ####
To do this, we can check if the name of the sprite is `"food"`.

_check if head collided with food_
```javascript
if(sprite.name === "food"){
  //code for steps 2 - 4 goes here
}
```

#### 2. get the snake sprite #### 
We'll need to get the snake sprite since we'll use it for adding a snake piece to the game.

_get snake sprite_
```javascript
var snake = Game.sprites.withName("snake");
```

#### 3. add a snake piece to the game ####

This is a X step process. First, we create a new snake piece using `SnakePiece` and assign it to a variable called `newSnakePiece`. 

_create a new snake piece_
```javascript
var newSnakePiece = new SnakePiece();
```

Then, we set the new snake pieces x and y position to the old x and y position of the last piece of the snake. The last piece of the snake is `snake.pieces[snake.pieces.length - 1]`.

_set position of new snake piece to old position of last piece of snake_
```javascript
newSnakePiece.position.x = snake.pieces[snake.pieces.length - 1].oldPosition.x;
newSnakePiece.position.y = snake.pieces[snake.pieces.length - 1].oldPosition.y;
```

Next, we add the new snake piece to the snake's array using `snake.pieces.push`.

_add new snake piece to snake's array_
```javascript
snake.pieces.push(newSnakePiece);
```

Last, we add the new snake piece to the game using `Game.sprites.add(<sprite>)`.

_add new snake piece to the game_
```javascript
Game.sprites.add(newSnakePiece);
```

Here's what it looks like all together.

```javascript
//create a new snake piece
var newSnakePiece = new SnakePiece();

//set x and y position to old x and y position of last snake piece
newSnakePiece.position.x = snake.pieces[snake.pieces.length - 1].oldPosition.x;
newSnakePiece.position.y = snake.pieces[snake.pieces.length - 1].oldPosition.y;

//add snake piece to the snake's array
snake.pieces.push(newSnakePiece);

//add snake piece to the game
Game.sprites.add(newSnakePiece);
```

#### 4. move food to random location ####
To move the food to a random location, we can assign a random value to the x and y position. The values should be between 2 and 18. This prevents the food from being placed outside or on top of the walls. To this, we'll use a a function developed by someone else that let's us generate a random number that's between two integers, `_.random(<low number>, <high number>)`, where `<low number>` is the lowest number that can be generated and `<high number>` is the highest number that can be generated.

```javascript
//set food to random position
sprite.position.x = _.random(2, 18);
sprite.position.y = _.random(2, 18);
```

Here's what everything looks like:

```javascript
//1. check if head collided with food
if(sprite.name === "food"){

  //2. get the snake sprite
  var snake = Game.sprites.withName("snake");

  //3. add a snake piece to the game
  //create a new snake piece
  var newSnakePiece = new SnakePiece();

  //set x and y position to old x and y position of last snake piece
  newSnakePiece.position.x = snake.pieces[snake.pieces.length - 1].oldPosition.x;
  newSnakePiece.position.y = snake.pieces[snake.pieces.length - 1].oldPosition.y;

  //add snake piece to the snake's array
  snake.pieces.push(newSnakePiece);

  //add snake piece to the game
  Game.sprites.add(newSnakePiece);

  //4. move food to random location
  //set food to random position
  sprite.position.x = _.random(2, 18);
  sprite.position.y = _.random(2, 18);
}
```

## wrap up ##
Congratulations! We made our second game: Snake. The full code can be found at [glitch.com](https://glitch.com/edit/#!/peach-justice). If you enjoyed creating games and would like to continue making games, here are some things to consider. First, you now have a strategy you can use and adjust for planning and developing your games.

  1. Think about the gameplay.  
  2. Think about what you'll need to setup in the framework.
  3. Think about your sprites' initial values and code.
  4. Think about your sprites' dynamic values and code.

Second, I would recommend learning and using a better supported framework for making games. The downside is that you'll need to spend some time learning a new framework. But the upside is that those frameworks have biggger communities and support for answering questions.

Here are some recommendations.

- Not necessarily programming, but still want to make more games? I recommend [GDevelop](https://gdevelop-app.com/). It has a visual drag-and-drop editor that let's you develop your games without having to write code.

- Have some experience modding games using Lua? I recommend [LOVE](http://love2d.org/). It will allow you to program your games using Lua.

- Want to learn how to make games using Java? I recommend [LibGDX](https://libgdx.badlogicgames.com/).

- Want to learn how to make games using Python? I recommend [PyGame](https://www.pygame.org/wiki/GettingStarted) or [Arcade](http://arcade.academy/).

- Want to learn how to make games using JavaScript? I recommend [Phaser](https://phaser.io/).

- Want to make games, but also need a level editor? I recommend [Unity](https://unity3d.com/) or [Godot](https://godotengine.org/). 
