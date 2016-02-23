---
layout: post
title:  "View holder misconceptions"
date:   2015-02-23 22:00:00
description: Android View Holder Pattern: I don't think it is what you think it is.
categories:
- patterns
- android
permalink: viewholder-misconception
---
Once in a while, I overhear collegues talking, or find internet discussions about the **View Holder pattern** in android.
If everything that is stated in these discussion was true, the View Holder would be a magical pattern solving several problems at once.
//TODO show some code!
In android the View Holder pattern is used to achive a better performance for such as `ListViews`<sup>*</sup>. That's it. Nothing to add. 
Sometimes however, there are some misconceptions about this pattern that I will clear up in this article.

## The View Holder is a pattern to reuse Views
That's the most popular misconception. Since every `ListView` tutorial in the world wide web also features the View Holder pattern, this conclusion comes fast.
The truth is that reusability is a feature - and actually the whole point - of the `ListView`.
Let's take a look at the `getView` method from the `Adapter` interface:
```java
 View getView(int position, View convertView, ViewGroup parent);
```
The first parameter is (as the name suggests) the position in the list. 

The second paramter, `convertView` is the important thing to achieve reusability. 
When implementing `getView` the naive way would be something like this:
```java
View getView(int position, View convertView, ViewGroup parent) {
  View v = View.inflate(parent.getContext(), R.layout.someView, parent);
    
  //do something with v
    
  return v;
}
```
As you probably know, this will cause the creation of a new `View` everytime the method is called. Which can be quite a lot on large lists.
To avoid this, the Android System gives passes us a `convertView`, so that we only inflate our `View` if the `convertView` is null:
```java
View getView(int position, View convertView, ViewGroup parent) {
  if(convertView == null) {
    convertView = View.inflate(parent.getContext(), R.layout.someView, parent);
  }
  //do something with convertView
    
  return convertView;
}
```
The system knows how many rows of our list can be visible at once. For each visible row, it passes a `null` as `convertView` into the `getView` method.
If the user scrolls the list and new rows become visible, the system fetches a non-`null` `contentView` from the other end of the list and passes it to the method. Because of the `convertView == null` check, no new `View` will be inflated.

This is the whole magic that happens to achieve reusability. The system knows whether a row is visible and can therefore pass an existing `View` object into the `getView` method, that we have to use.

##Without the View Holder, contents of the rows get messed up 
It's a common error when dealing with `ListViews`: When scrolling, the whole row or only parts of the row is suddenly duplicated in the other (just appearing) rows. See e.g. [this StackOverflow question](http://stackoverflow.com/questions/20637978/android-listview-row-repeating-item).
While using a View Holder helps with this issue by defining a single place of accessing the row's `Views`, it is not the absense of the Holder that causes the problem. The real problem is the developer's slopiness. 
Consider this code:
```java
View getView(int position, View convertView, ViewGroup parent) {
  if(convertView == null) {
    convertView = View.inflate(parent.getContext(), R.layout.someView, parent);
  }
  
  if(getItem(position).randomNumber == -1){
    ((TextView)convertView.findViewById(R.id.text)).setText("Number is invalid!");
  }
    
  return convertView;
}
```
While everything looks good when the `ListView` is visible at first, scrolling will quickly spawn "Number is invalid!" texts in every single row.

Because of the reuse

<sup>*</sup>I'm using `ListView` in this example, but it's actually true for all `AdapterView`s
