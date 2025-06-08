---
title: "Fragment: getContext vs requireContext"
date: 2018-08-26T12:00:00-04:00
categories:
  - Tech
tags:
  - Kotlin
  - Java
  - Android
  - Fragment
---

Android team annotated some sdk methods with NonNull and Nullable in support library in 27.1.0. That change leads to warnings and errors like on the picture above.

**TLDR: use requireContext only when you are sure fragment is attached to its host(onResume, onViewCreated, etc).**

## The problem

![getContext vs requireContext](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aj3bcgbd1yptiv9mbwrc.png)


## Why did they do this?

At first sight it looks like adding more and more pain to android apps development. But actually this annotations help us to decrease crash rate.

getContext can return null when fragment isn’t attached to its host. Let’s take a look at two examples.

```java
@Override
public void onResume() {
    super.onResume();
    Resources resources = getContext().getResources();
}
```

First sample shows us that as long as fragment is attached to its host(in onResume for example) it’s safe to use getContext without nullability checking.

```java
@Override
public void onResume() {
    super.onResume();
    new Handler().postDelayed(new Runnable() {
        @Override
        public void run() {
            final Resources resources = getContext().getResources();
        }
    }, TimeUnit.SECONDS.toMillis(5));
}
```

The second one reveals potential NullPointerException. If after 5 seconds fragment isn’t attached to host, app is crashed.

So getContext Nullable annotation can prevent bugs like in the second sample. But it causes a useless warning in the first sample in Java and an error in Kotlin.

## How should we fix this?

Firstly we should fix potentials bugs.

1. Find all places where fragment can be detached at some point of time
2. Change code like in the sample below

In Java:

```java
Context context = getContext();
if (context == null) {
    // Handle null context scenario
} else {
    // Old code with context
}

```

In Kotlin:

```kotlin
val context: Context = this.context ?: return // or if block
```

Finally replace the rest with requireContext() which returns NonNull Context instead of getContext(). But keep in mind that requireContext() will throw an exception if fragment isn’t attached to its host. So you should use requireContext() with respect to fragment lifecycle.

## Wrong fixes

In Java:

```java
1.
// requireContext() does the same
Context context = getContext();
if (context == null) {
    throw new IllegalStateException("");
}

2.
// Warning supressing only hides the real problem
private void doSomething(@NonNull final Context context) {
}

@Override
public void onResume() {
    super.onResume();

    //noinspection ConstantConditions
    doSomething(getContext());
}
```

In Kotlin:

```kotlin
// It is similar to requireContext() but with NullPointerException
val context: Context = context!!
```

## Other similar methods

1. getActivity vs requireActivity
2. getHost vs requireHost
3. getFragmentManager vs requireFragmentManager
