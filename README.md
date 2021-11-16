# Technical-Test

## The Level


#### Actions

**WASD/Space** - Movement

**TAB**- Open/Close the inventory 

**R**- While dragging an item, pressing R will rotate the item.

**Drag**- Dragging within the inventory will allow the item to be moved to the new spot

**Drop**- Dragging an item outside the inventory will drop it in front of the player.


#### Restrictions

As each item takes up a set amount of space within the inventory, players are restricted from carrying too many large items.


#### Spawning

There are buttons on the ground that when walked on, spawn an item provided there is not an existing item.

Within the level there are 3 areas for different types of items.



* Storage items which change the inventory size.
* Weapons that take up a lot of inventory size.
* Standard items that are small.


## Item Representation

Item information is kept within the **ItemObject** object, it includes information for:



* Icon
* Tile Size (size inside inventory)
* Type - Standard, Weapon, Storage. For different logic on pickup
* Class - For spawning


#### Icons


	Content/_Items/_ItemRendering

Items are displayed in the inventory using a **3DMeshCapture** which has a SceneCaptureComponent2D to capture the Base_Item’s mesh as a 2D image.


#### TileSize


	Content/_Inventory/InventorySize

The item's tile size is kept as an **Int Point**, however I created a struct called **InventorySize** for easier readability.


#### Type


	Content/_Inventory/ItemType

An enum containing the item types, to allow for different functions based on the type.

For example on collision a Storage type will update the inventory size rather than being added to the inventory.


#### Class


	Content/_Items/

Items are displayed within the world as a derivative of the **Base_Item** class which handles collision, material & rotation logic.


## Item Spawning


	Content/_Items/_Spawning

**ItemSpawner_Component** handles the logic for spawning items, it has 2 functions.



* **AttemptToSpawn**, which will consider if there is a current item spawned and if not, will **SpawnItem.**
* **SpawnItem** will spawn a given class at a location.

**ItemSpawnerButton** has an ItemSpawner_Component and will spawn an item when walked upon. It also visually displays the collision process by changing colours.


## Inventory System


### Logic


#### Inventory_Component


	Content/_Inventory/Inventory_Component

The component contains a list of all the **ItemObjects** within it (**Items**), this is stored as a standard array, with 2 functions that can convert between the array index and the 2D tile location. Because the items take up multiple tiles in the inventory they are stored at multiple locations within the array.

When items are added, moved or removed from the inventory, it will use the top left tile (TopLeftIndex) to handle all the placement logic.

When picking up an item, it will **TryAddItem**, which checks if there is enough room using **IsRoomAvailable** and if that returns true, it will **AddItemAt** the first available space

**IsRoomAvailable** requires the item object's tile size and the TopLeftIndex and will then loop through all the tiles that would be needed to place the item within the inventory and check if they are valid or occupied.

**AddItemAt** takes in an ItemObject and the TopLeftIndex and then sets all related tiles to be occupied with the ItemObject.

**GetItemAtIndex** returns the item at a provided index, as long as the item is valid.

**RemoveItem** will remove an item from the array and set all the related tiles to unoccupied.

**GetAllItems** creates a map of all the distinct items from within the Items array. The key is the ItemObject and the value is the TopLeftIndex.

**UpdateSize** will change the size of the inventory. When a change event happens, it caches the current inventory, clears it, resizes and then attempts to re-add each item back into the inventory. If there is no space, it will drop the item.

**ForEachIndex** returns the tile locations that an item will occupy.

**IsTileValid** checks if the tile location is within the grid bounds.

**ForEachItem** will loop through each distinct item.


### Display


#### Inventory_Widget


	Content/_Inventory/Inventory_Widget	

This widget handles most of the display logic for the Inventory_Component. It has 2 children that handle the main parts of the display: 

**ItemInventory_Widget** - Item icons 

**BackgroundGrid_Widget** - The background grid.

The event **Refresh** is called whenever the Inventory_Component is modified. It clears and then rebuilds the background and item widgets.

<span style="text-decoration:underline;">Visual</span>

The size of the grid elements is customisable with these variables:

**_TileSize_**- Changes the size of the background grid and the item icons.

**_BorderSize_**- Changes the spacing between the grid items, is also considered when creating the item icons.

**TileSizeToActualSize** takes in an ItemObject’s size and converts it to the size used by the Item_Widget.

**AddItemToInventory** creates an Item_Widget and places it within the ItemInventory_Widget’s grid panel at a calculated location.

<span style="text-decoration:underline;">Dragging</span>

**OnDragOver** sets the variable **_DraggedItemTopLeftTile_** by calculating which tile it is over as well as which sector of the tile it is within.

When dragging, **OnPaint** is called to create a box that displays if the position the ItemObject is over is valid or not. 

**RotateDragItem** will rotate the held ItemObject.

**OnDrop** will either place the held ItemObject within the inventory or drop the item on the ground. It also contains a way for an item to be dropped at an invalid location, but still be added to the inventory provided there is space.


#### ItemInventory_Widget


	Content/_Inventory/ItemInventory_Widget

This widget is just a holder for the item icons.


#### Item_Widget


	Content/_Inventory/Item_Widget

This widget is created from an ItemObject and will display its icon. It is resizable based on the given TileSize and BorderSize. It is also what is moved during the drag drop operation.


#### BackgroundGrid_Widget


	Content/_Inventory/BackgroundGrid_Widget

Creates a grid of **GridTile_Widgets**, with inputs for TileSize & BorderSize.

Has a bool **_IsOverInventory_** for the drag & drop logic within the **Inventory_Widget.**

Also processes the mouse’s position while over the grid for use in item placement.


#### GridTile_Widget


	Content/_Inventory/GridTile_Widget

When constructed it will set itself to the TileSize.


#### HUD_Widget


	Content/_Inventory/HUD_Widget

Used to display the text in the top right corner.

<span style="text-decoration:underline;">Other</span>

**OnMouseButtonDown_0 **is used on both the** Inventory_Widget** and **BackgroundGrid_Widget** to prevent the player's camera from being moved whilst the inventory is open.

**Rarity** is an enum created to let the items have different materials depending on their rarity. This only applies to Storage & Weapon Item Types.

All 3D assets were downloaded from https://craftpix.net/freebies/

