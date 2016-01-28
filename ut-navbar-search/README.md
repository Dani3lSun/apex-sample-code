#Oracle APEX - UT Navbar Search
This tutorial shows you how to implement a global search input inside the navigation bar of APEX UT theme.
All steps doesn´t require you to change theme templates or the theme JS/CSS files itself, thus you don´t lose your current theme subscription and stay update save...

The steps includes these todo´s:
- create a placeholder navigation bar entry
- create a region and text item on global page (page 0)
- create a "onload" DA on global page (moving the item to navbar and adding some nice animations)
- add some CSS to UT theme roller
- create a application process for redirecting to other pages (global page branch)
- optional: create a DA on global page for a loading spinner if enter key gets pressed


##Changelog
####1.1 - solved IE and Firefox Navbar link issues

####1.0 - Initial Release



##1 - Create a placeholder navigation bar entry
Go to your Application --> Shared Components --> Navigation Bar list --> "Desktop Navigation Bar" and create a new list entry. For best looking results choose the smallest display sequence so the list entry stays on the left side.

```
Sequence: 1
Image/Class: fa-search navbar-search
List Entry Label: &nbsp;
Target type: URL
URL Target: javascript:document.getElementById('P0_SEARCH').focus();
```

![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/navbar_entry.png)

####Explanation
These settings add a new navbar list entry with a search icon and a blank label which sets the focus on click on our input item we create in step 2. The custom class "navbar-search" is also required later to move the item using javascript to this position.


##2 - Create a region and text item on global page (page 0)
Go to your global page (usually page 0) and create a new region and a text field. Choose a position there other regions are not disrupted...
```
Region:
Position: Footer
Title: Global Search
Template: Blank with Attributes

Item:
Position: Item position of region
Name: P0_SEARCH
Type: Text Field
Submit when Enter pressed: Yes
(Label) Template: Hidden
Width: 30 characters
```

![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/p0_region_item.png)

####Explanation
These settings add a new blank region with a position usually not visible on other pages. The region includes our text field which we move to navbar in the next step.


##3 - Create a "onload" DA on global page (moving the item to navbar and adding some nice animations)
Go to your global page (usually page 0) and create a new Dynamic Action (page load) using the code below:

```
Name: globalSearch
Event: Page Load
True Action: Execute JavaScript
Fire On Page Load: Yes
```

Code:
```javascript
// move element to navbar
function moveItem2Navbar(pItem) {
  var element = $('#' + pItem).detach();
  $('.navbar-search').append(element);
}
// fade out animation
function fadeOutItem(pItem, pWidth, pTime) {
  $('#' + pItem).animate({
    width: pWidth,
    backgroundColor: "#fff",
    color: "#000"
  }, pTime);
}
// fade in animation
function fadeInItem(pItem, pWidth, pTime) {
  $('#' + pItem).animate({
    width: pWidth,
    backgroundColor: "transparent",
    color: "#fff"
  }, pTime);
}
// set inititial attributes
moveItem2Navbar('P0_SEARCH');
$s('P0_SEARCH', '...');
// on focus fade out
$('#P0_SEARCH').focus(function() {
  $s('P0_SEARCH', '');
  fadeOutItem('P0_SEARCH', 280, 300);
});
// on focosout fade in
$('#P0_SEARCH').focusout(function() {
  $s('P0_SEARCH', '...');
  fadeInItem('P0_SEARCH', 25, 300);
});
```

![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/p0_onload_da.png)

####Explanation
This javascript code moves the item from our blank region to the navigation bar (appends item to our custom css class) and creates some nice animations for fading out and back in...


##4 - Add some CSS to UT theme roller
Run your application and open UT Theme Roller from developer bar. Under Custom CSS part add this:

```
#P0_SEARCH {
  width: 25px;
  height: 25px;
  border-width: 0px;
  border-radius: 3px !important;
  background-color: transparent;
  color: #fff;
  margin-top: -6px;
  margin-bottom: -4px;
  margin-left: 5px;
}
.t-Header-navBar {
  margin-bottom: -5px;
}
.t-NavigationBar {
  margin-bottom: 5px;
}
```

####Explanation
This CSS adds some default style to our input item and navigation bar. If you don´t like it or you already customized your UT theme, change the styles so that fits best into your application.


##5 - Create a application process for redirecting to other pages (global page branch)
Go to Your Application --> Shared Components --> Application Processes --> Create

```
Name: GLOBAL_SEARCH_PAGE_BRANCH
Process Point: On Submit: After Page Submissions - Before Computations and Validations
Type: PL/SQL Anonymous Block
Condition Type: Request = Expression 1
Expression 1: P0_SEARCH
```

Code:
```language-sql
DECLARE
  l_url VARCHAR2(1000);
  --
BEGIN
  -- url
  l_url := apex_page.get_url(p_application => :app_id,
                             p_page        => '100', -- your search result page
                             p_clear_cache => '100', -- clear cache search result page
                             p_items       => 'P100_SEARCH', -- target page search item
                             p_values      => REPLACE(:p0_search,
                                                      ',',
                                                      '')); -- our page 0 search item without comma
  htp.init;
  -- redirect
  owa_util.redirect_url(l_url);
  apex_application.stop_apex_engine;
END;
```

![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/app_process.png)

####Explanation
This Application Process gets executed if we submit our P0_SEARCH item (enter pressed) and redirects to another page (for example a search result page) and sets a item on target page.


##6 - Optional: Create a DA on global page for a loading spinner if enter key gets pressed
Go to your global page (usually page 0) and create a new Dynamic Action using the code below:

```
Name: globalSearchShowSpinner
Event: Key Down
Selection Type: Item P0_SEARCH
Condition:
Javascript Expression: this.browserEvent.which == 13
True Action: Execute JavaScript
Fire On Page Load: No
```

Code:
```javascript
// apex loading spinner
apex.util.showSpinner();
// suppress soft keyboard (touch devices)
setTimeout(function(){
  $('#P0_SEARCH').blur();
},200);
```

![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/p0_keypress_da.png)

####Explanation
This DA adds a loading spinner to the page until target page is up. On touch devices the soft keyboard will fade out...


##Preview
![](https://raw.githubusercontent.com/Dani3lSun/apex-sample-code/master/ut-navbar-search/images/preview.gif)
---
