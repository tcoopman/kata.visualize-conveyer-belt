# kata.visualize-conveyer-belt

Implement the following 2 functions:

* `eventsToVisualization(List[Event]) -> Visualization`
* `visualizationToEvents(Visualization) -> List[Event])`

## Domain

Given these events:

```
ConveyerInitialized(belt: Belt)
ItemAdded(item: Item)
ItemEnteredStation(item: Item, station: Station, position: Position)
ItemLeftStation(item: Item, station: Station)
Stepped
Paused
Resumed
```

With

```
Station(name: String, size: Int)
Belt(length: Int, stations: List((Position, Station)))
Item(name: String)
Position: Int
```

## Visualization

Here is how a visualization looks like:

### a Belt

`_`: an empty position on a belt.

For example: `Belt(5, [])` looks like: `_ _ _ _ _` (there is always a space for clarity)

So, in our functions this would look like this:

```
events = [ConveyorInitialized(belt=Belt(size=1, stations=[]))]
visualization = eventsToVisualization(events)
assert visaulization == "_"
```

and

```
visualization = "_"
events = visualizationToEvents(visualization)
assert events == [ConveyorInitialized(belt=Belt(size=1, stations=[]))]
```

From now on for brevity we will always write this as:

```
Events: [ConveyorInitialized(belt=Belt(size=1, stations=[]))]
Output: "_"
```

When you see this, you should write a test that checks your function from `events -> visualization` and from `visualization -> events`

### Stations and Items

`S(a)` is a Station with name a and size 1
`SSS(b)` is a Station with name b and size 3

`I(a)` is an Item with name a

## Some initial examples

You can assume that all examples are valid

Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[]))]`
Output: `_ _ _`

Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[])), ItemAdded(item=Item(name="a"))]`
Output: `I(a) _ _`

Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[(0, Station(name="s", size=1))]))]`
Output: `S(a) _ _` 

Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[(0, Station(name="s", size=3))]))]`
Output: `SSS(a)`

## New rules

### Stepping

`Stepped` means: the belt moved one position to the right. All items one the belt will move 1 to the right.

Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[])), ItemAdded(item=Item(name="a")), Stepped, Stepped]`
Output: `_ _ I(a)`

If an item steps of the belt, the item is just removed

```
Events: `[ConveyorInitialized(belt=Belt(size=3, stations=[])), ItemAdded(item=Item(name="a")), Stepped, Stepped, Stepped]`
Output: `_ _ _`
```

### Enter and leaving stations.

You can make the following assumption:
* items always enter the station at position 0 (position relative to the station) and leave the station at the same position.
* when any item is in a station, the belt can't be moved.

`S(a[I(i)])`: A station with name a, of size 1 is processing item i
We would have an event: ItemEnteredStation(Item(i), Station(a), 0)

`SS[I(i)]S(a)`: A station with name a, of size 3 is processing item i on position 1
We would have an event: ItemEnteredStation(Item(i), Station(a), 1)

*warning*:
`S(a)I(i)` and `S(a) I(i)` are not the same.
The first is we have a station and item on the same position. But the item left the station (processing by the station was done): `ItemLeftStation(Item(i),Station(a))`.
For `S(a) I(i)` we have an additional `Stepped` event.

The second one: the station is at position 0 and the item on position 1.

```
item = Item(name="i")
station = Station(name="a", size=2)
Events: [ConveyorInitialized(belt=Belt(size=4, stations=[(0, station))])), ItemAdded(item), ItemEnteredStation(item, station, 0), ItemLeftStation(ite, station)]
Output: "SS(a)I(i) _ _"
```

```
item = Item(name="i")
station = Station(name="a", size=2)
Events: [ConveyorInitialized(belt=Belt(size=3, stations=[(0, station))])), ItemAdded(item), ItemEnteredStation(item, station, 0), ItemLeftStation(ite, station), Stepped]
Output: "SS(a) I(i) _"
```

## Adding paused and resumed.

* You must emit a `Paused` when you go from no items in a station to at least one item in a station.
* You must emit a `Resumed` when you go from some items in a station to no items in a station.

complex example:

```
station1 = Station(name="s1", size=1)
station2 = Station(name="s2", size=2)
belt = Belt(size=4, stations=[(1, station1), (2, station2)])
item1 = Item(name="i1")
item2 = Item(name="i2")

Events: `[
  ConveyorInitialized(belt),
  ItemAdded(item1),
  Stepped,
  ItemEnteredStation(station1, item1, 0),
  Paused,
  ItemAdded(item2)
  ItemLeftStation(station1, item1),
  Resumed,
  Stepped,
  ItemEnteredStation(station1, item2, 0),
  ItemEnteredStation(station2, item1, 0),
  Paused
]`
Output: `_ S(s1[I(i2)]) S(s2[I(i1)]) _`
```

If multiple items enter a station at the same step, always assume that the events are emitted from left to right.

The previous example continued:

```
station1 = Station(name="s1", size=1)
station2 = Station(name="s2", size=2)
belt = Belt(size=4, stations=[(1, station1), (2, station2)])
item1 = Item(name="i1")
item2 = Item(name="i2")

Events: `[
  ConveyorInitialized(belt),
  ItemAdded(item1),
  Stepped,
  ItemEnteredStation(station1, item1, 0),
  Paused,
  ItemAdded(item2)
  ItemLeftStation(station1, item1),
  Resumed,
  Stepped,
  ItemEnteredStation(station1, item2, 0),
  ItemEnteredStation(station2, item1, 0),
  Paused
  ItemLeftStation(station1, item2),
  ItemLeftStation(station2, item1),
  Resumed
]`
Output: `_ S(s1)I(i2) S(s2)I(i1) _`
```


