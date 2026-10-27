# S04-Code-Smells-CMPM-121

Max Marinescu
Zhizhuo Xu
Michael Pu Yang

# Duplicated Code 

## Introduction

### What is it?

A Duplicated Code smell is the smell of the same or similar structures of code appearing in more than one place. It is the first smell in Fowler's catalog and can come in many forms:

| Type | Description | Example |
|------|-------------|---------|
| **Exact copy** | Identical text in two or more places | The same discount formula pasted into two functions |
| **Near copy** | Same structure, but with different literals, names, or small tweaks | Two shipping cost calculators that differ only in rates |
| **Structural / hierarchy duplication** | Sibling classes that implement the same behavior independently | `Salesman` and `Engineer` each defining the same `Paycheck` method |
| **Knowledge duplication** | Different-looking code that encodes the same *rule* | A validation rule in both the UI and the API layer |

### Why does it happen?

Duplication normally doesn't come from bad intentions. Most of the time duplicated code occurs wherever we're human, when we're under pressure or our brains are thinking too fast for our hands to type. Sometimes it's best to let the ideas flow out of you while they can, and you can make things look pretty after. 

Some common reasons why we may write Duplicate Code:

- **Copy-paste deadline pressure.** Copying a working function and editing a few values is the fastest way to ship features. Something we're all a victim of. Fowler describes exactly this: copy the function, change the values, and you've created duplicated code (p. 50).
- **Parallel work.** Two developers solve the same problem independently, in different files or branches.
- **If it ain't broke, don't fix it.** Nobody wants to change a shared function that others rely on, so they clone it.
- **Skipped cleanup.** The feature works throughout the program, so the refactoring step is skipped. Fowler's **Rule of Three** says: *"Three strikes and you refactor"* (p. 50). Copying once is tolerable, but a third copy means it's time to consolidate.

### Why is it a problem?

Fowler explicitly states (p.72): every time you read the copies, you need to read them carefully to see if there’s any difference and if you need to change the duplicated code you have to catch every copy. This leads to a lot of time wasted by programmers when reading through repositories, and a lot more unnecessary bulk code. With so many independent copies it can also lead to a breeding ground for bugs, where it's difficult to keep track of so many different variables.

Commmon problems that arise:

- **Inconsistent bug fixes.** You fix the bug in one copy and ship the same bug from another.
- **Higher reading cost.** Readers must differentiate copies in their heads to spot the subtle differences.
- **Bloat.** More code means more to test, review, and understand. As Fowler puts it, eliminating duplication ensures the code "says everything once and only once" (p. 47).
- **Hidden divergence.** Copies drift apart over time, and nobody remembers which differences were intentional.

The idea is also known as the [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) ("Don't Repeat Yourself"). Removing duplication won't make the program run faster; the payoff is that the code becomes much easier to modify correctly.

## Prevention and Refactoring

Fowler's listed fixes for the Duplicated Code smell is **[Extract Function](https://refactoring.com/catalog/extractFunction.html)** (p. 106), **[Slide Statements](https://refactoring.com/catalog/slideStatements.html)** (p. 223), and **[Pull Up Method](https://refactoring.com/catalog/pullUpMethod.html)** (p. 350). Fowler explains situations to use each, for example **Extract Function** can be used where you see the same expression or code block in more than one place. 

### Some Common Prevention Practices

1. **Follow the Rule of Three.** Tolerate one copy, wince at the second, and refactor at the third.
2. **Keep functions small and well named.** Small, intention-revealing functions are easy to reuse, so people call them instead of rewriting them.
3. **Maintain good tests.** A solid test suite removes the fear that makes people copy code instead of changing it.
4. **Review for duplication.** Put "does this already exist?" on your pull-request checklist.

### How to Refactor

1. Differentiate copies to find out exactly what differs. 
2. If code is scattered, use **Slide Statements** to bring associated code together. 
3. Use **Extract Function** to pull out shared systems. 
4. Turn differences into necessary parameters, or use **Pull Up Method** to pull shared behavior up in class hierarchies. 
5. Replace duplicated code with a call, testing after small changes so that behavior is preserved. 

## Code Examples

### Example 1 (Python): Extract Function

**The Smell.** This code exhibits 3 Mobs with different base stats depending on which Mob is declared. They independently print each of their base stats such as Name, Health, Strength, Durability, etc. 

```python
# Set of mob classes with different base stats
slime = Mob("Slime")
for stat in slime:
    print(f"{stat.name}: {stat.value}")
    
goblin = Mob("Goblin")
for stat in goblin:
    print(f"{stat.name}: {stat.value}")

mimic = Mob("Mimic")
for stat in mimic:
    print(f"{stat.name}: {stat.value}")
```

**What's wrong with it.** Each Mob uses similar loops to print their stats, leaving bulky duplicate code. We can clean up our codebase and make it easier to read by extracting a function that prints stats for a Mob.

**The Fix.**
```python
def printStats(mob: Mob) -> None:
    for stat in mob:
    print(f"{stat.name}: {stat.value}")

# Set of mob classes with different base stats
slime = Mob("Slime")
goblin = Mob("Goblin")
mimic = Mob("Mimic")

printStats(slime)
printStats(goblin)
printStats(mimic)
```

Now we have a parameterized print function that can print stats based on the calling mob. The function also allows us to add new mobs in the future without having to worry about implementing a way to print their stats. This also shows whoever is reading our codebase that this section prints stats for all of our mobs without having to read through each individual loop.

### Example 2 (JavaScript): Pull Up Method

**The Smell.** This code shows an example of a class that handles Mario powerups. The Powerup class has subclasses for a Fire Flower or Ice Flower respectively, and each subclass contains its own method for returning its unique character model.

```js
class Powerup {...}

class FireFlower extends Powerup {
  get powerUpModel() {...}
}

class IceFlower extends Powerup {
  get powerUpModel() {...}
}
```

**What's wrong with it.** 
Both subclasses repeat the same `powerUpModel` getter, which is duplicate behavior. If the naming rule changes later, it must be updated in multiple places. This also provides a weakpoint for bugs when calling functions with similar names. 

**The Fix.** 
```js
class Powerup {
  get powerUpModel() {
    return this._model;
  }
}

class FireFlower extends Powerup {}
class IceFlower extends Powerup {}
```

Move the shared getter into the base class so each subclass inherits it instead of reimplementing it. This allows each powerup to call a retrieval of their model,  no matter the specific type it is. If there are features from subclasses that are not on the superclass, we may need to use **[Pull Up Field](https://refactoring.com/catalog/pullUpField.html)** (p. 353).

# Loops

## Introduction

### What is it?

The Loops smell is when a plain `for` or `while` loop does work that a pipeline of operations like `filter` and `map` could express more clearly. Loops have been around since the earliest languages, but Fowler argues they've become outdated, like old fashion trends (p. XX).

### Why does it happen?

Mostly, history. When the first edition of *Refactoring* came out, Fowler and Beck already disliked loops, but languages like Java didn't offer a better option. Now that first-class functions are widely supported, that excuse is gone (p. XX). Habit also plays a part, since loops are one of the first things we learn.

### Why is it a problem?

A loop mixes the "what" with the "how," so you have to read the whole body to understand it. Pipeline operations make it easier to see which elements are processed and what happens to them (p. XX). Loops also need temporary variables, and they can quietly grow to do several unrelated jobs.

## Prevention and Refactoring

Fowler's main fix is **[Replace Loop with Pipeline](https://refactoring.com/catalog/replaceLoopWithPipeline.html)** (p. 231): figure out what the loop is doing (filtering, transforming, totaling) and swap in the matching pipeline operation. If one loop has several jobs, **[Split Loop](https://refactoring.com/catalog/splitLoop.html)** is a good first step.

To prevent it, ask yourself before writing a loop: "Am I filtering, transforming, or totaling?" If so, there's probably a built-in for it. Keep each loop to one job, and keep pipelines short.

## Code Examples

### Example 1 (JavaScript): Replace Loop with Pipeline

**The smell.** This loop collects the names of enemies that are still alive:

```js
const aliveNames = [];
for (const enemy of enemies) {
  if (enemy.health > 0) {
    aliveNames.push(enemy.name);
  }
}
```

**What's wrong with it.** You have to read the whole body to see that the `if` is a filter and the `push` picks one field. It also needs a temporary array.

**The fix.**

```js
const aliveNames = enemies
  .filter(enemy => enemy.health > 0)
  .map(enemy => enemy.name);
```

Now it reads like a sentence: "take the enemies, keep the ones with health left, get their names."

### Example 2 (Python): the same idea in another language

**The smell.**

```python
emails = []
for order in orders:
    if order.total > 100:
        emails.append(order.customer_email)
```

**The fix.** Python's version of a pipeline is the list comprehension:

```python
emails = [order.customer_email for order in orders if order.total > 100]
```

Same refactoring, same benefit: no temporary list, and the intent is clear at a glance.
Split Variables
