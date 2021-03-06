# FancyAdapters
A collection of customizable RecyclerView Adapters for Android, that provide various functionality like item selection, contextual action mode controls, drag&amp;drop and swiping, among other.

# Installation
In your project's `build.gradle` add jitpack's repository:
```gradle
allprojects {
	repositories {
		...
		maven { url "https://jitpack.io" }
	}
}
```
And in your module's `build.gradle` add the following dependency:
```gradle
dependencies {
    ...
	compile 'com.github.mikiloz92:fancyadapters:0.2.0'
}
```

# Usage

Even though understanding and using the following adapters isn't really a straightforward process, I can guarantee you that the results are going to be worth it! So, without further due, let's take a look at the basics of the adapters:

## SelectableAdapter

**SelectableAdapter** is the simplest of them all. It just provides item selection functionality, combined with contextual action mode controls.

![alt text](https://github.com/MikiLoz92/FancyAdapters/blob/master/art/selectable_adapter.gif?raw=true "SelectableAdapter")

### Extending SelectableAdapter

**SelectableAdapter** is an abstract class, and that means you have to implement some of its functionalities on your own. Let's take a look at everything you need to implement and we'll explain step by step how to do it and what your code will be doing:

``` java
public class CustomAdapter extends SelectableAdapter<String, CustomAdapter.CustomViewHolder> {

    public CustomAdapter(List<String> items, RecyclerView recyclerView) {
        super(items, recyclerView);
    }

    @Override
    public CustomViewHolder onCreateSelectableViewHolder(ViewGroup parent, int viewType) { }

    @Override
    public void onBindViewHolder(CustomViewHolder holder, int position) { }

    @Override
    public void onItemSelected(CustomViewHolder holder, int position) { }

    @Override
    public void onItemDeselected(CustomViewHolder holder, int position) { }

    public class CustomActionModeCallback extends AdapterActionModeCallback {
    
        @Override
        public boolean onCreateActionMode(ActionMode mode, Menu menu) { }
  
        @Override
        public boolean onPrepareActionMode(ActionMode mode, Menu menu) { }
  
        @Override
        public boolean onActionItemClicked(ActionMode mode, MenuItem item) { }
        
        @Override
        public void onExitActionMode(ActionMode mode) { }
      
    }

    @Override
    public ActionMode startActionMode() { }

    @Override
    public void updateActionMode(ActionMode mode, int selectedCount) { }

    public class CustomViewHolder extends RecyclerView.ViewHolder {

        public CustomViewHolder(View itemView) {
            super(itemView);
        }
        
    }
    
}
```

First, notice that when extending from SelectableAdapter, you should provide a type for the elements your adapter will be storing, in this case ```String```s.

* `onCreateSelectableViewHolder`: This method creates a ViewHolder and returns it. It is called each time the RecyclerView needs a new ViewHolder, because either it doesn't have or it ran out of ViewHolders to recycle.
* `onBindViewHolder`: This method is used, generally, to replace the elements of a ViewHolder that was just recycled for the elements of a ViewHolder that is needed at the moment (this element corresponds to the `position` parameter). Furthermore, you should check if the ViewHolder that is being binded is selected or not (use the public method `isSelected(int position)`) and call `onItemSelected` or `onItemDeselected` accordingly.
* The `onItemSelected` and `onItemDeselected` methods should take care of activating or deactivating any view that provides visual feedback of the selection state of an item.
* We need to define an AdapterActionModeCallback child. This child contains the methods that will be called on every event of the ActionMode. Regarding the titles of the methods to override, this is pretty self-explanatory:
  * The `onCreateActionMode` is called whenever the intention of creating an ActionMode is made public. We should inflate any possible menu here, and perform any actions on the currently active (by active I mean "not recycled") ViewHolders, if needed. You can access them with the `viewHolders` public list. Watch the sample app to find out more about manipulating the ViewHolders programmatically.
  * The `onPrepareActionMode` method should set the title and perform any remaining actions once the ActionMode is created.
  * The `onActionItemClicked` method responds to action item click events.
  * The `onExitActionMode` is called when the ActionMode is being destroyed, be it because we clicked the back button or because we deselected the last item that remained selected. As with the `onCreateActionMode` method, any actions on the currently active ViewHolders should be performed here as well, if needed.
* The `startActionMode` method should create and return an ActionMode using an instance of the callback defined earlier. For the purpose of creating the ActionMode, you should employ any feasible strategy. For example, if you are extending the SelectableAdapter inside of an Activity class, this is easy: just call `startSupportActionMode()`. However, if you are doing it inside of a Fragment class, it gets a little trickier, you have to first access the parent Activity to call the method.
* The `updateActionMode` method should update the ActionMode title when the number of items selected changes.
* Finally, a ViewHolder type needs to be provided to the SelectableAdapter. Within this ViewHolder's constructor, you should define any OnClickListener, OnLongClickListener or any input listener that triggers the selection mode, toggles, selects or deselects an item. For these four events, you should call `triggerSelectionMode()`, `toggleItem()`, `selectItem()` and `deselectItem()`, respectively.

## SuperSelectableAdapter

**SuperSelectableAdapter** adds drag&drop and swiping capabilities to **SelectableAdapter**.

![alt text](https://github.com/MikiLoz92/FancyAdapters/blob/master/art/super_selectable_adapter.gif?raw=true "SuperSelectableAdapter")

### Extending SuperSelectableAdapter

There are few more requisites to extend SuperSelectableAdapter than extending its parent class:

* Implement the `onMove` method: this method is called whenever a ViewHolder that you are currently dragging forces another ViewHolder to swap its position. You should perform any backend data movement here, or at least move the items from the `items` ArrayList of SuperSelectableAdapter.
* Implement the `onSwipe` method: this method is called whenever a ViewHolder is swiped. Any possible backend data removal or modification should be done here.
* As with SelectableAdapter, you can customize the ViewHolder behavior on input events by using the `triggerSelectionMode()`, `toggleItem()`, `selectItem()` and `deselectItem()` methods. Just one more thing left: you can call the `dragStart()` method whenver you want to start a drag. Generally, you would do this on your ViewHolder itemView's OnLongClickListener, but you could also add a *handle* View to your ViewHolder and start the drag when that *handle* is touched. The possibilities are endless!
* By default, an item will be selected when you start a drag on its ViewHolder and drop it on the same spot without moving it from there, as you can see on the gif image above. However, you can disable this behavior by using the `setTriggerSelectionModeByDroppingViewHolder(boolean triggerOnDrop)` method.

## SelectableViewAdapter

**SelectableViewAdapter** is a SuperSelectableAdapter that provides visual feedback of the selection status of an item. What makes it different from the adapter that it extends from is that there are two methods in the ViewHolder (it must extend from SelectableViewAdapter.ViewHolder), `getSelectableViewID` and `getSelectedIndicatorResourceID` that provide, respectively:

1. The ID (R.id.\*) of the View that will act as a selectable View (in the gif below, it is the ID of the TextView that marks the row position (on the left of each item). This is considered the **selectableView**.
2. The layout ID (R.layout.\*) from which to inflate a View that will replace the View just described (in the gif below, it is a View that has tick mark). This is considered the **selectedIndicatorView**.

![alt text](https://github.com/MikiLoz92/FancyAdapters/blob/master/art/selectable_view_adapter.gif?raw=true "SuperSelectableAdapter")


## HandleAdapter

**HandleAdapter** is a special type of adapter. It sacrifices a little bit of customizability in order to bring to the user an easier to implement adapter. The adapter bases its behavior in the idea that ViewHolders can only be moved by dragging a certain View, called the *handle* View. It the gif below, it is represented by the image on the left of each row.

![alt text](https://github.com/MikiLoz92/FancyAdapters/blob/master/art/handle_adapter.gif?raw=true "SuperSelectableAdapter")
