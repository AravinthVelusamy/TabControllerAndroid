# TabControllerAndroid *by Appolica*
This library provides an easy to use API for switching between fragments that share the same container. Implementing tabs, such as the ones Instagram has at the bottom of the layout, is a much simpler goal to achieve when using **TabController**.<br>
Tabs can be switched by using either show/hide or attach/detach. You can even provide your own implementation of the way tabs are being shown.

The library is developed and maintained by [Appolica](http://www.appolica.com).

<img src="https://github.com/Appolica/TabControllerAndroid/blob/master/doc_resources/gif/sample.gif?raw=true" height="500">

*Note: The sample uses [BottomBar](https://github.com/roughike/BottomBar) in order to give a nice example of using this library. TabController is for managing fragments and does not provide any UI views.*

## Download
[ ![Download](https://api.bintray.com/packages/appolica-ltd/maven/TabControllerAndroid/images/download.svg) ](https://bintray.com/appolica-ltd/maven/TabControllerAndroid/_latestVersion)

## Gradle
```gradle
dependencies {
    ...
    compile 'com.appolica:tab-controller:1.0.3'
}
```

## Example

There are two ways of using this library. One is to use `TabControllerFragment`.  It encapsulates all the fragments and their container into a single fragment. 

You can use it like any other fragment by adding it to the xml layout:
```xml
<fragment
        android:id="@+id/fragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        class="com.appolica.tabcontroller.fragment.TabControllerFragment"/>
```

or instantiating it and adding it by yourself:

```java
final TabControllerFragment controllerFragment = new TabControllerFragment();
getSupportFragmentManager()
                .beginTransaction()
                .add(conainerId, controllerFragment)
                .commitNow();
```

Obtain the TabController instance simply like this:
```java
final TabController tabController = controllerFragment.getTabController();
```

---
The other way is to provide your own fragment container. If you choose to use this, you have to create the TabController instance by yourself:

```java
final TabController tabController = new TabController(getSupportFragmentManager(), R.id.container);
```
---

It doesn't matter whether you use your own container or TabControllerFragment. Once you obtain a TabController instance, everything is the same.

In order to show your fragment you have to use `TabController::switchTo` method. This method accepts an implementation of `FragmentProvider`.
Suppose you have multiple fragments and the first one is called `HomeFragment`. Then you would need something like `HomeFragmentProvider`, that implements `FragmentProvider`.

```java
public class HomeFragment extends Fragment {
    public static final String TAG = "HomeFragment";

    public static HomeFragment getInstance(int tabNum) {
        final HomeFragment fragment = new HomeFragment();

        return tab;
    }

    ...
}
```
```java
class HomeFragmentProvider implements FragmentProvider{
    @Override
    public String getTag() {
        return HomeFragment.TAG;
    }

    @Override
    public Fragment getInstance() {
        return HomeFragment.getInstance();
    }
}
```
Create an instance of `HomeFragmentProvider` and pass it to `TabController::switchTo` in order to show your `HomeFragment`.
```java
final HomeFragmentProvider homeProvider = new HomeFragmentProvider();
tabController.switchTo(homeProvider);
```
The `TabController` will create a new instance (by calling `FragmentProvider::getInstance`) of your fragment if it hasn't been already created. This depends on whether `FragmentManager::findFragmentByTag` returns `null` for the tag, given by `FragmentProvider::getTag`.

## API

#### TabController

##### Constructors:
* `TabController(FragmentManager fragmentManager, int containerId)` - Create a new instance of TabController with the default ShowHideHandler.
* `TabController(FragmentManager fragmentManager, int containerId, ShowHideHandler showHideHandler)` - Create a new instance of TabController.

##### Public methods:
* `void switchTo(FragmentProvider provider)` - Show the given fragment in the container, provided to the constructor. If `FragmentManager::findFragmentByTag` returns null for the tag, given by the provider, your fragment's instance will be obtained by calling `FragmentProvider::getInstance`. Otherwise it will be reused. If there is already a visible fragment, it will be hidden. How fragments are shown/hidden depends on ShowHideHandler.
* `Fragment getVisibleFragment()` - Iterates through the fragments, returned by FragmentManager::getFragments. Returns the first visible fragment found in the list or null.
* `Fragment getFragment(@NonNull FragmentProvider fragmentProvider)` - Find a fragment by it's `FragmentProvider`. Same as calling fragmentManager.findFragmentByTag(fragmentProvider.getTag());
* `void save(Bundle savedInstanceState)` - Save the state of the `TabController` in order to be able to restore your last visible fragment when your app restores.
* `void restore(@Nullable Bundle savedInstanceState)` - Restore the state of the `TabController`. This will show the last visible fragment before saving the state.
* `void setChangeListener(OnFragmentChangeListener changeListener)` - Set listener to be notified on one of the `TabController`'s events.

#### ShowHideHandler
Implementations of this interface determine how your fragments are going to be shown/hidden. It could be by using `FragmentTransaction.show(Fragment)`/`FragmentTransaction.hide(Fragment)` or `FragmentTransaction.attach(Fragment)`/`FragmentTransaction.detach(Fragment)`.

##### Public methods:
* `FragmentTransaction show(FragmentTransaction transaction, Fragment fragment)` - Show the given fragment within the given transaction the way you want.
* `FragmentTransaction hide(FragmentTransaction transaction, Fragment fragment)` - Hide the given fragment within the given transaction the way you want.
* `void save(Bundle saveControllerState, Fragment fragment)` - Called when TabController is saving its state. In some cases (like when you show/hide your fragment by using `FragmentTransaction.show/hide`) you may want to save the visibility of your fragment. This is where you should do that. This method is called for each fragment, returned from `FragmentManager.getFragments()`.
* `void restore(@Nullable Bundle savedControllerState, FragmentTransaction transaction, Fragment fragment)` - Called when `TabController` is restoring its state. In some cases (like when you show/hide your fragment by using `FragmentTransaction.show/hide`) in order to restore the visibility of your fragments you should implement this method. Your fragment will become visible or hidden depending on what action you add to the given `FragmentTransaction`. This method is called for each fragment returned from `FragmentManagegetFragments()`.
* `boolean isVisible(Fragment fragment)` - Since Fragment.isVisible() and Fragment.isHidden() work different, `TabController` is using this abstract method, relying that the implementation will work properly.

#### FragmentProvider
Used by `TabController`. Implementation of this interface should provide a tag and an instance of the fragment that will be shown/hidden by the controller.

##### Public methods:
* `String getTag()` - Provide the fragment's tag.
* `Fragment getInstance()` - Provide fragment's instance.

#### OnFragmentChangeListener
Pass this listener to `TabController.setChangeListener(OnFragmentChangeListener)` to be notified for one of it's events.

##### Public methods:
* `void onFragmentShown(FragmentProvider provider, Fragment shownFragment)` - Called when one of your fragments has been shown.
* `void onFragmentAlreadyVisible(FragmentProvider provider, Fragment visibleFragment)` - Called on attempt to show an already visible fragment.
* `void onFragmentCreated(FragmentProvider provider, Fragment createdFragment)` - Called when the fragment you want to show has been created by the TabController.

#### TabControllerFragment
An encapsulation for the fragments that are managed through the `TabController`. All of these fragments are switched between within this fragment using child fragment manager.<br>
We encourage you to use this fragment in order to restrict the access of the `TabController` to any other fragments, that aren't related to it and vice versa.

##### Public methods:
* `TabController getTabController()` - Obtain the `TabController`. Its instance is created in `Fragment.onViewCreated(View, Bundle)`.







# License

Copyright 2017 Appolica Ltd.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
