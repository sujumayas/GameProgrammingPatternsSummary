# Game Programming Patterns

This is a summary of http://gameprogrammingpatterns.com/command.html#directions-for-actors

<3 All the love <3

## Command

Command (or Reify)

>"Encapsulate a request as an object, thereby letting users parameterize clientes with different requests, queue or log requests, and support undoable operations" Gang of Four

It means taking some concept and turning it into a piece of *data* - an object - that you can stick in a variable, pass to a function, etc. So, by saying the Command pattern is a 'reified method call', what we mean is that it's a method call wrapped in an object. 

Some examples = 'Callbacks', 'First class functions', 'Function pointer', 'closure' and 'partially applied functions'. 

>"Commands are an object-oriented replacement for callbacks" Gang of Four

### Configure Input

 Instead of doing this: 

```c++
void InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) jump();
  else if (isPressed(BUTTON_Y)) fireGun();
  else if (isPressed(BUTTON_A)) swapWeapon();
  else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```

We can decouple the method calls (jump / fireGun / luchInnefectively) trough a Command Pattern and make them all extend the Command class. 

We can create a  Command Class that calls a method execute on an actor: 

```c++
class Command
{
public: 
	virtual ~Command(){}
	virtual void execute(GameActor& actor) = 0;
};
```

Then we can create all the derived command that will invoke methods on the actor passed. We override the execute to call a method in the actor. (Every single derived command will do the same, wrapping the actor.x into a class called XCommand that you can call later)

```c++
class JumpCommand : public Command
{
public:
  virtual void execute(GameActor& actor)
  {
    actor.jump();
  }
};
```

Then, we can use this one class to make any character in the game hop arround. We only need to map this to an inputhandler:

```c++
class InputHandler
{
public:
  void handleInput();

  // Methods to bind commands...
  // Like bind x to jump, a to 

private:
  Command* buttonX_;
  Command* buttonY_;
  Command* buttonA_;
  Command* buttonB_;
};
```
This will let us map different button keys to different commands (out of this scope, but cool!!) , and then we can do like this:

```c++
Command* InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) return buttonX_;
  if (isPressed(BUTTON_Y)) return buttonY_;
  if (isPressed(BUTTON_A)) return buttonA_;
  if (isPressed(BUTTON_B)) return buttonB_;

  // Nothing pressed, so do nothing.
  return NULL;
}
```

This will not execute the the command, but will return it like a callback or function reference. 

So now, finally, we can handle everything from inside the actor:

```c++
// This can be executed every frame (on update)
// we use the input handler that will return a command :D
Command* command = inputHandler.handleInput(); 
if (command)
{
  command->execute(actor);
}
```

This is fucking awesome. 

Depending on the mechanics of your game, you can also pass coordinates (mouse input) or anyother thing you think your commands will always use (inside the scope of your game) and will never change. Remember "dont patternize something that will never change".

The best part of this, is that we can actually map the inputhandling differently for AI and use the same pattern/classes/code for handling any AI movements in the game. This could even map AI movement logic to the player itself and do our demo. 

For that to work we will need to implement an Event Queue to handle the "stream" of commands that the AI will input into the actor is gonna move. 

Also, a well-known use case for this pattern, is when you create a command that will not be used directly (its execute() method), but it will be instantiated every time you use it, so you can save a reference to it, and have more than 1 (execute()) method in it. This is used a lot in "undo" moves, in strategy turn based games. So you can have a queue of commands (that execute), and then have a queue of saved last commands that are available to undo. lovely. 

### Functional Example

For example, if we were building a game in JavaScript, we could create a move unit command just like this:

```javascript
function makeMoveUnitCommand(unit, x, y) {
  // This function here is the command object:
  return function() {
    unit.moveTo(x, y);
  }
}
```

We could add support for undo as well using a pair of closures:

```javascript
function makeMoveUnitCommand(unit, x, y) {
  var xBefore, yBefore;
  return {
    execute: function() {
      xBefore = unit.x();
      yBefore = unit.y();
      unit.moveTo(x, y);
    },
    undo: function() {
      unit.moveTo(xBefore, yBefore);
    }
  };
}
```

If you’re comfortable with a functional style, this way of doing things is natural. If you aren’t, I hope this chapter helped you along the way a bit. For me, the usefulness of the Command pattern really shows how effective the functional paradigm is for many problems.

### See Also

You may end up with a lot of different command classes. In order to make it easier to implement those, it’s often helpful to define a concrete base class with a bunch of convenient high-level methods that the derived commands can compose to define their behavior. That turns the command’s main execute() method into the **Subclass Sandbox** pattern.

In our examples, we explicitly chose which actor would handle a command. In some cases, especially where your object model is hierarchical, it may not be so cut-and-dried. An object may respond to a command, or it may decide to pawn it off on some subordinate object. If you do that, you’ve got yourself the ****Chain of Responsibility** pattern.

Some commands are stateless chunks of pure behavior like the JumpCommand in the first example. In cases like that, having more than one instance of that class wastes memory since all instances are equivalent. The **Flyweight** pattern addresses that.



















