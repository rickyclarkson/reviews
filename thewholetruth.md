# The truth, the whole truth and nothing but the truth

## The truth

A function signature can tell the truth, or lie. Here's one that lies:

Java: ```<T> T getInstance()```

Kotlin: ```fun <T> getInstance(): T```

Haskell: ```getInstance :: t```

How does it lie? It says we can get an arbitrary T/t, from nothing, when in all of these languages there is not enough information to make a T.

The Java implementation can return null, or throw an unchecked exception. The Kotlin implementation can throw an exception. The Haskell version can do something unsafe and crash, but there isn't a sane implementation of this function, as its signature is a lie.

So let's incrementally explore what truth might look like.

## The whole truth

Here's a limited version of truth:

Java: ```String enrich(String value)```

Kotlin: ```fun enrich(value: String): String```

Haskell: ```enrich :: String -> String```

There, we can enrich a String and get a String back, presumably richer somehow. But that's not the whole truth, unless we'd be happy with the identity function as the implementation:

Java: ```String enrich(String value) { return value; }```

Kotlin: ```fun enrich(value: String): String = value```

Haskell: ```
enrich :: String -> String
enrich value = value  -- could also use enrich = id
```

Let's say that the result is actually not just an arbitrary string, but one of a closed set, or empty if not matched:

Java: ```
String enrich(String value) {
  return switch (value) {
    case "CD", "DVD", "BLUERAY" -> "ROTATING DISC",
    case "SD CARD", "USB STICK" -> "SOLID STATE",
    default -> ""
  }
}
```

Kotlin: ```
fun enrich(value: String): String = when (value) {
  "CD", "DVD", "BLUERAY" -> "ROTATING DISC"
  "SD CARD", "USB STICK" -> "SOLID STATE"
  else -> ""
}
```

Haskell: ```
enrich :: String -> String
enrich "CD" = "ROTATING DISC"
enrich "DVD" = "ROTATING DISC"
enrich "BLUERAY" = "ROTATING DISC"
enrich "SD CARD" = "SOLID STATE"
enrich "USB STICK" = "SOLID STATE"
enrich _ = ""
```

It's now becoming more apparent too that enrich just categorizes storage types, so we can introduce a couple of types:

Java: ```
enum StorageType { CD, DVD, BLUERAY, SD_CARD, USB_STICK }
enum StorageCategory { ROTATING_DISK, SOLID_STATE }
```

Kotlin: ```
enum class StorageType { CD, DVD, BLUERAY, SD_CARD, USB_STICK }
enum class StorageCategory { ROTATING_DISK, SOLID_STATE }
```

Haskell: ```
data StorageType = Cd | Dvd | Blueray | SdCard | UsbStick
data StorageCategory = RotatingDisk | SolidState
```

Now enrich can be better defined, and better named as categorize, telling the whole truth:

Java: ```
StorageCategory categorize(StorageType type) {
  return switch (type) {
    case CD, DVD, BLUERAY -> StorageCategory.ROTATING_DISK
    case SD_CARD, USB_STICK -> StorageCategory.SOLID_STATE
  }
}
```

Kotlin: ```
fun categorize(type: StorageType): StorageCategory = when (type) {
  StorageType.CD, StorageType.DVD, StorageType.BLUERAY -> StorageCategory.ROTATING_DISK
  StorageType.SD_CARD, StorageType.USB_STICK -> StorageCategory.SOLID_STATE
}
```

Haskell: ```
categorize :: StorageType -> StorageCategory
categorize Cd -> RotatingDisk
categorize Dvd -> RotatingDisk
categorize Blueray -> RotatingDisk
categorize SdCard -> SolidState
categorize UsbStick -> SolidState
```

## And nothing but the truth

If we later need to add a StorageType that doesn't map well to a StorageCategory, and for some reason we cannot add a StorageCategory - that type is owned by another team and they need to evaluate priorities before adding enum members - let's see what our options are.

Java jumps ahead, by allowing us to create a NoMatchingStorageType exception, and throwing that. We can even declare it in the function signature:

```StorageCategory categorize(type: StorageType) throws NoMatchingStorageType { .. Notebook -> throw new NoMatchingStorageType(type); }```

Kotlin lets us throw the exception, and even specify it via @Throws, but it won't make sure the caller handles it. This feels a bit weak, so it's more common to make the function return a nullable type:

```fun categorize(type: StorageType): StorageCategory? = when (..) { .. Notebook -> null }```

Haskell just doesn't have exceptions, or null, so we'll return a Maybe - similar to a Java Optional:

```
categorize :: StorageType -> Maybe StorageCategory  -- read as 'function that takes a StorageType and returns a Maybe that might hold a StorageCategory'
categorize Cd -> Just StorageCategory  -- Just constructs a Maybe that holds a value
[snip]
categorize Notebook -> Nothing  -- Nothing is a Maybe that doesn't hold a value
```

Practically, working with Java checked exceptions or null is painful enough that we'll want a similar approach to Haskell's, even in Java:

```Optional<StorageCategory> categoize(type: StorageType) { .. }```

Unfortunately nothing in the signatures in any of these languages prevents an (un)checked exception from being thrown, so calling categorize without reading its implementation is a bit of a leap of faith. It has the ability to throw exceptions, even though the function signature doesn't allow that.

