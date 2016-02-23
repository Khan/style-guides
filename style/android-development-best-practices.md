## Prefer Immutability

Our principal goal as programmers is to reduce complexity. At any point in a program, we can reason about the state of a constant trivially -- its state is the state upon assignment. By contrast, the state of a variable can change endlessly.
 
With judicious use, immutable objects can lead to quicker execution speed and lower memory usage. The hash value of a String, the query parameters of an immutable URL, and the distance traced by an immutable sequence of points are all immutable as well. We can cache their values for later use instead of recompute them on every access.

An immutable value is safe in many ways. We can share an immutable value with clients without worry that the client will silently mutate the value. Similarly, a client can accept an immutable value as a parameter without needing to defensively copy it. An immutable value is also thread-safe, because a client must acquire a lock only to read data that another client can concurrently modify. An immutable value renders that moot.

To summarize: Immutable values help tame complexity.

A good approach for creating immutable types is to define types that are simply data, and which do not have behavior. Typically any such added behavior will seek to modify the class, and we want to avoid this. For example, consider the `SecondsWatched` class, which tracks the seconds watched values in a video:

```java
public final class SecondsWatched {
  public final long lastSecondWatched;
  public final long totalSecondsWatched;

  public SecondsWatched(long lastSecondWatched, long totalSecondsWatched) {
    /* Preconditions checks here... */

    this.lastSecondWatched = lastSecondWatched;
    this.totalSecondsWatched = totalSecondsWatched;
  }

  /* Method equals, hashCode, and toString follow here. */
}
```

Note that this instance is immutable because its `lastSecondsWatched` and `totalSecondsWatched` fields are `final`, and hence cannot be reassigned.

The `VideoUserProgress` instance composes a `SecondsWatched` instance of this class:

```java
public final class VideoUserProgress {
  public final ContentItemIdentifier videoItemId;
  public final SecondsWatched secondsWatched;
  public final Optional<Date> lastWatchedDate;

  /* Constructor, equals, hashCode, and toString follow here. */
}
```

Again, a `SecondsWatched` instance is immutable. So is each instance of the `ContentItemIdentifier` class, as well each instance of the `Date` class. A constructed `Optional` cannot be reassigned to refer to another (possibly `null`) value, and so it is immutable as well. By virtue of the `VideoUserProgress` composing instances of immutable classes, and making those fields `final`, then each `VideoUserProgress` instance is immutable as well. Composition of immutable instances yields more immutable instances.

To assist with defining such immutable classes, we typically use AutoValue, which is discussed later.

## Prefer interfaces over implementations

The Java standard library has a rich suite of interfaces for collections: `List`, `Map`, `SortedMap`, `Set`, and `SortedSet`. The standard implementations of these interfaces include `ArrayList`, `LinkedList`, `HashMap`, `TreeMap`, `HashSet`, and `TreeSet`.

For their parameters and return values, method signatures should refer to the most generalizable collection type that is most appropriate. For example, consider the `VideoSubtitleSequence` class, which represents a transcript for a video:

```java
public final class VideoSubtitleSequence {
  private final ImmutableList<VideoSubtitle> subtitles;

  public VideoSubtitleSequence(List<VideoSubtitle> subtitles) {
    this.subtitles = ImmutableList.copyOf(subtitles);
  }

  ...
}
```

A transcript is an ordering of `VideoSubtitle` instances. Its constructor has a parameter type of `List`. It does not specify a list implementation, such as `ArrayList`:

```java
public VideoSubtitleSequence(ArrayList<VideoSubtitle> subtitles) { ... }
```

This constructor could not be invoked if the client had a value of type `List`, `LinkedList`, or some other `List` implementation. The client would have to create and populate a temporary `ArrayList` simply for the purposes of calling this method.

Likewise, the constructor does not choose a superinterface of `List`, such as `Collection` or `Iterable`:

```java
public VideoSubtitleSequence(Collection<VideoSubtitle> subtitles) { ... }
```

Because `Set` extends `Collection`, a client could construct a `VideoSubtitleSequence` with a `Set` of `VideoSubtitle` instances. But the iteration order for a `Set` implementation is undefined. Therefore the order in which the client adds `VideoSubtitle` instances to the set is not guaranteed, or even likely, to equal the iteration order when the video plays.

## Use Google Guava

From the Google Guava web page:

> The Guava project contains several of Google's core libraries that we rely on in our Java-based projects: collections, caching, primitives support, concurrency libraries, common annotations, string processing, I/O, and so forth.

Some of the packages are indispensable for disciplined Java development:

### Package `com.google.common.base`

#### Class `Preconditions`

Use methods of the `Preconditions` class in every constructor you define to ensure that a client is constructing a valid instance. By catching such invalid data when the instance is created, we can determine the source of the invalid parameters by navigating the stacktrace. If we checked for validity upon use, we not only put the burden on every client to ensure that the data is valid, and this precludes finding where the invalid data comes from. We primarily use two of its static methods: `checkNotNull` and `checkArgument`.

The `checkNotNull` method accepts a parameter and throws an `NullPointerException` if it is `null`. It returns that parameter if it is not. For example, the `FeaturedContentViewPagerAdapter` constructor uses `checkNotNull` to ensure that it is constructed with non-`null` members:

```java
public FeaturedContentViewPagerAdapter(final Context context,
                                       final DeviceInfo deviceInfo,
                                       final ObservableContentDatabase contentDatabase) {
  mContext = checkNotNull(context);
  mDeviceInfo = checkNotNull(deviceInfo);
  mContentDatabase = checkNotNull(contentDatabase);

  mLayoutInflater = LayoutInflater.from(mContext);
}
```

Method `checkNotNull` should not only be used in constructors, but on parameters of setters that may modify the state of an instance after it is constructed.

The `checkArgument` method asserts that its expression is `true`. If not, it throws an `IllegalArgumentException`. It's best to provide a second parameter that provides a more detailed message for the `IllegalArgumentException`. This message should include any parameter in the expression that evaluated as `false`. This makes debugging easier. For example, method `getResizedUrl` ensures that an `HttpUrl` is not constructed with a negative value for the image width:

```java
public HttpUrl getResizedUrl(final int longestWidth) {
  checkArgument(longestWidth > 0, "Parameter longestWidth too small: " + longestWidth);
  checkArgument(longestWidth <= APP_ENGINE_IMAGE_RESIZE_MAX_SIDE_LENGTH_PX,
          "Parameter longestWidth too large: " + longestWidth);

  return baseUrl
      .newBuilder()
      .encodedPath(baseUrl.encodedPath() + "=s" + longestWidth)
      .build();
}
```

#### Class `Optional`

In Java, we typically represent an "absent" value as `null`. For example, given a `Map` to videos from their corresponding unique identifiers, we may have:

```java
final Video video = videosById.get(videoId);
renderVideo(video);
```

However, `Map` will return `null` if no value with the corresponding key exists. If this is the case, then later attempting to access a property of the `Video` instance will throw a `NullPointerException`. To warn the reader that the `Video` instance may be `null`, we can precede its type with the `@Nullable` annotation:

```java
final @Nullable Video video = videosById.get(videoId);
if (video != null) {
  renderVideo(video);
}
```

However, it is easy to forget to add the `@Nullable` annotation. Moreover, the following code is still legal, which could throw a `NullPointerException` if method `renderVideo` expects a `Video` instance instead of `null`:

```java
final @Nullable Video video = videosById.get(videoId);
renderVideo(video);
```

By contrast, more modern languages may encode *optionality* of values in the type system. In Swift, for example, a type suffixed with `?` can assume `nil`, while a type that stands alone cannot:

```swift
let r: String = "foo"
let o1: String? = nil
let o2: String? = "bar"

let e: String = nil // compilation error
```

Variable `r` has a type of `String`, and so it must contain a string value. It cannot contain `nil`. Variable `o1` and `o2` have a type of `String?`, and so they may contain an optional string, or alternatively `nil`. (In this case, `o1` contains `nil`, while `o2` contains a string.) The last statement, which attempts to assign `nil` to variable `e`, fails to compile, because its type is `String` and such a type cannot assume a `nil` value.

Given a variable of type `String?` that actually refers to a string, we can safely access that value by "unwrapping" it:

```swift
if let s = o2 {
  // o2 is type String?, but s is type String
  print(s)
}
```

Code within the block can refer to `s`, which of type `String` and thus cannot refer to `nil`.

Back in Java, the `Optional` class from Guava is a library-level imitation of this concept. We can use `Optional` to encode the optionality of such values in the type system. This helps ensure that clients deal with such optional values in a direct manner. For example:

```java
final Optional<Video> videoOptional = getVideoById(id);
if (videoOptional.isPresent()) {
  final Video video = videoOptional.get(); 
  renderVideo(video);
}
```

The `Optional` instance above may contain a `Video` instance if such an instance exists with the given identifier. In that case, we say that the `Video` instance is *present*, not *absent*. Its `isPresent` method therefore returns `true`, and the call to `get` returns the `Video` instance.

Note that the following code will not compile:

```java
final Optional<Video> videoOptional = getVideoById(id);
renderVideo(videoOptional);
```

This is because `videoOptional` is of type `Optional<Video>`, and the `renderVideo` method requires a `Video` parameter. Therefore using `Optional` forces us to reckon with the optionality of values.

#### Implementing `toString`

Some classes may not have value semantics and so the use of AutoValue may be unsuitable. Nonetheless it may be useful to implement `toString` so that logging the value yields useful information. The `MoreObjects` class has a useful `toStringHelper` method that, as its name implies, helps construct a `toString` value. For example:

```java
public final class ByteCountingSink extends ForwardingSink {
  private long mNumBytesWritten;
  private long mNumBytesTotal;

  ...

  @Override
  public String toString() {
    return MoreObjects.toStringHelper(this)
        .add("numBytesWritten", mNumBytesWritten)
        .add("numBytesTotal", mNumBytesTotal)
        .toString();
  }
}
```

If a `ByteCountingSink` instance has a `mNumBytesWritten` value of `123` and a `mNumBytesTotal` value of `456`, then invoking its `toString` method will return:

```
ByteCountingSink={numBytesWritten=123, numBytesTotal=456}
```

#### Implementing `toString` with inheritance

If there is a base class, it may be helpful for it to provide a `protected` factory method that creates a `ToStringHelper` instance and adds to it the base class fields. The derived class implementation of `toString` can then invoke this base class factory method and add to it the derived class fields before returning. For example, consider the abstract base class `OkHttpDownloadState`:

```java
abstract class OkHttpDownloadState<T extends DownloadableResource<?>> {
  final T resource;

  // ...

  protected MoreObjects.ToStringHelper getToStringHelper() {
      return MoreObjects.toStringHelper(this)
              .add("resource", resource);
  }
}
```

The `toString` implementation of subclass `DownloadedOkHttpDownloadState` adds its own fields to the `ToStringHelper` before returning the constructed string:

```java
final class DownloadedOkHttpDownloadState<T extends DownloadableResource<?>>
    extends OkHttpDownloadState<T> {
  final File file;

  // ...

  @Override
  public String toString() {
    return super.getToStringHelper()
        .add("file", file)
        .toString();
  }
}
```

Other subclasses can implement `toString` similarly.

### Package `com.google.common.annotations`

#### Annotation `VisibleForTesting`

We typically annotate a package-private variable, method, or type with `@VisibleForTesting` when that variable would typically be private, but we are increasing its scope to package-private only so that a test in the same package (but in the parallel source tree for tests) can have access to the annotated element.

For example, `VideoSubtitleJsonDecoder` is a class that consumes JSON from a `JsonReader` and constructs a `VideoSubtitle` instance from the consumed values. Its inner class `JsonKeys` serves as a namespace for all constants defining names in the JSON.

```java
public final class VideoSubtitleJsonDecoder {
  @VisibleForTesting
  static final class JsonKeys {
    static final String START_TIME = "startTime";
    static final String TEXT = "text";
  }

  public static VideoSubtitle read(final JsonReader reader) throws IOException {
    // ... Read values with names START_TIME and TEXT.
  }
}
```

Typically we would specify `private` visibility for `JsonKeys`, since the expected keys are an implementation detail that clients of a `VideoSubtitleJsonDecoder` should be unconcerned with. But by making the keys package-private and annotating them with `VisibleForTesting`, the test named `VideoSubtitleJsonDecoderTest` can construct a JSON object and ensure that it is decoded correctly:

```java
final JsonObject json = new JsonObject();
json.addProperty(JsonKeys.TEXT, text);
json.addProperty(JsonKeys.START_TIME, startTime);
final JsonReader jsonReader = new JsonReader(new StringReader(json.toString()));

final VideoSubtitle subtitle = VideoSubtitleJsonDecoder.read(jsonReader);
assertEquals(text, subtitle.text);
assertEquals(startTime, subtitle.startTime);
```

In other places, we use `VisibleForTesting` to provide an alternate constructor for testing. For example, a `RecentlyWorkedOnDatabase` instance stores the most recently worked on content items, such as videos or articles, that the user has worked on. The application constructs it using its public constructor, which limits the number of stored content items to 10. But we provide an alternate, package-private constructor where the content items can be specified:

```java
public class RecentlyWorkedOnDatabase implements Closeable {
  private static final int DEFAULT_MAX_NUM_RECENTLY_WORKED_ON_ITEMS = 10;

  private final Database mDatabase;
  private final int mMaxNumRecentlyWorkedOnItems;

  public RecentlyWorkedOnDatabase(final Database database) {
      this(database, DEFAULT_MAX_NUM_RECENTLY_WORKED_ON_ITEMS);
  }

  @VisibleForTesting
  RecentlyWorkedOnDatabase(final Database database, final int maxNumRecentlyWorkedOnItems) {
    checkArgument(maxNumRecentlyWorkedOnItems > 0,
        "Invalid maxNumRecentlyWorkedOnItems: " + maxNumRecentlyWorkedOnItems);

    mDatabase = checkNotNull(database);
    mMaxNumRecentlyWorkedOnItems = maxNumRecentlyWorkedOnItems;
  }

  // ... Remainder of implementation follows here.
}
```

This facilitates testing the functionality where the least recently worked on content items are evicted whenever the maximum number of items would otherwise be exceeded.

### Package `com.google.common.collect`

### Immutable Collections

As described earlier, immutability is a critical aspect of controlling complexity in our programs. As such, for each of the collection interfaces `List`, `Map`, `SortedMap`, `Set`, and `SortedSet`, Guava defines an implementation whose instances are immutable. This is only possible because the mutating methods of each collection interface, such as `add`, `set`, or `remove`, are defined as optional operations. The underlying implementation can choose to implement the method as specified by the interface, or throw an `UnsupportedOperationException`. The collections defined by Guava choose the latter.

> The standard libraries for some languages, such as Objective-C and Scala, instead define separate types for mutable and immutable collections.

##### Construction from individual elements

To construct an immutable collection from individual elements, invoke the correct overload form of its `of` static factory method:

```java
public static final List<ResultColumn> ALL_COLUMNS = ImmutableList.of(
    CONTENT_KEY,
    Videos.VIDEO_SIZE,
    Videos.VIDEO_STATUS,
    Videos.TRANSCRIPT_SIZE,
    Videos.TRANSCRIPT_STATUS
);
```

##### Construction from another collection

To construct an immutbale collection with the contents of an collection, such as an `Iterable` or an array, call the correct overload of its `copyOf` static factory method:

```java
public final class NodeTree {
  public final List<NodeTableEntity> nodes;
  public final Set<NodeToNodeTableEntity> relationships;

  public NodeTree(final List<NodeTableEntity> nodes, final Set<NodeToNodeTableEntity> relationships) {
    this.nodes = ImmutableList.copyOf(nodes);
    this.relationships = ImmutableSet.copyOf(relationships);
  }

  ...
}
```

##### Construction with a builder

Finally, if you cannot statically construct such an immutable instance, use an instance of its corresponding [builder](https://en.wikipedia.org/wiki/Builder_pattern) class. Such an instance is mutable and accumulates elements. Calling `build` constructs an immutable instance with those accumulated elements:

```java
public static List<VideoSubtitle> readSubtitles(JsonReader reader) throws IOException {
  ImmutableList.Builder<VideoSubtitle> builder = ImmutableList.builder();

  reader.beginArray();
  while (reader.hasNext()) {
    builder.add(VideoSubtitleJsonDecoder.read(reader));
  }
  reader.endArray();

  return builder.build();
}
```

##### Defensive copying

A class should defensively copy any collection instance that it is injected with. For example, consider the `JavaScriptCommand` class:

```java
public final class JavaScriptCommand {
  public final String methodName;
  public final List<Parameter> parameters;

  public JavaScriptCommand(String methodName, List<Parameter> parameters) {
    this.methodName = Preconditions.checkNotNull(methodName);
    this.parameters = ImmutableList.copyOf(parameters);
  }

  ...
}
```

If a client constructs a `JavaScriptCommand` instance, and then mutates the `List` instance that it passed in as the `parameters` value, then the `JavaScriptCommand` instance will not reflect those mutations. By defensively copying, it enforces its immutability.

Note that the `copyOf` method above will use reflection to test whether its parameter is also an `ImmutableList` instance. If so, then it simply returns its parameter instead of constructing a new `ImmutableList` instance. This is because we must only make defensive copies to guard against unexpected mutations, but immutable instances by definition do not afford that.

Consequently, the following test will pass:

```java
List<String> original = ImmutableList.of("foo", "bar");
List<String> copy = ImmutableList.copyOf(original);

assertSame(original, copy);
assertTrue(original == copy);
```

##### Guarding aginst `null`

Finally, `ImmutableList` and `ImmutableSet` prohibit `null` elements, while `ImmutableMap` prohibits `null` keys or values. Attempting to construct such an instance will throw a `NullPointerException`. If you require an `ImmutableMap` that permits `null` values, we have an `ImmutableNullableMap` instance. But before building a `List` or `Set` analogue, thoroughly consider alternative designs.

#### Using `Lists` and `Maps` and `Sets`

Classes like `ImmutableList`, `ImmutableMap`, and `ImmutableSet` are helpful for constructing new immutable collections. By contrast, classes like `Lists`, `Maps`, and `Sets` provide static utilty methods pertaining to their respective interfaces. They also provide many static factory methods for creating instances of standard implementations. For example, the `newArrayList` method returns an `ArrayList` populated with the elements of the given iterator, and avoids use of the diamond operator on the right side of the assignment statement:

```java
ArrayList<SerializedStackEntry> stack = Lists.newArrayList(mSerializedBackStack.descendingIterator());
```

Class `Lists` also has a `transform` method that creates a new `List` that is a transformed view of a given list:

```java
List<Download.Progress> allChildDownloadProgress = Lists.transform(
    allChildDownloads,
    childDownload -> childDownload.progress
);
```

Class `Maps` has utility methods for `Map` instances. For example, its `uniqueIndex` method returns an `ImmutableMap` with the given elements as values, each with a key that is generated by applying the supplied function:

```java
final Map<ContentItemIdentifier, ? extends BookmarkDownloadEntity> downloadEntities = Maps.uniqueIndex(
    downloadEntitiesList,
    entity -> entity.contentItemIdentifier()
);
```

Class `Sets` has utility methods for `Set` instances. For example, the `intersection` method returns an unmodifiable view of the intersection of two `Set` instances:

```java
Set<ContentItemIdentifier> downloadedContentItems =
    Sets.intersection(downloadEntities.keySet(), fileSizesByContentId.keySet());
```

Similarly, there are utility classes for other standard collection interfaces in the Java library:

* `Iterables` for `Iterable` implementations
* `Iterators` for `Iterator` implementations
* `Queues` for `Queue` implementations

Moreover, there are even utility classes for collection interfaces defined by Guava:

* `Multimaps` for `Multimap` implementations
* `Multisets` for `Multiset` implementations

Before you attempt writing your own utility method for a collection, find its corresponding class of static utility methods, and check whether the method you intend to write already exists.

#### Using `FluentIterable`

If you have a collection of values, and need to yield another collection by filtering or transforming those values, then use `FluentIterable`. This allows you to chain together the sequence of operations in a straight-line fashion, as opposed to creating an unreadable nesting of calls to achieve the same effect:

```java
Set<ContentItemIdentifier> downloadedContentItemIds = FluentIterable
    .from(mBookmarkDatabase.fetchAllBookmarksSortedByAscendingDate())
    .filter(bookmarkEntity -> {
      if (bookmarkEntity.downloadEntity().isPresent()) {
        return bookmarkEntity.downloadEntity().get().progress.isComplete();
      } else {
        return false;
      }
    })
    .transform(BookmarkEntity::identifier)
    .toSet();
```

Note that instead of materializing the resulting elements in an collection, you can invoke one of its many methods to use those elements as input to a provided function:

```java
String searchFilterQueryParameter = FluentIterable.from(searchFilterDomains)
    .transform(SearchApi::slugForSearchFilter)
    .join(COMMA_JOINER);
```

## Use AutoValue

We use [AutoValue](https://github.com/google/auto/tree/master/value) from Google to assist with writing classes with value semantics. For such classes, object identity is irrelevant, and instances are considered interchangeable based only on the equality of their field values. This requires implementing `equals` and `hashCode` in a repetitive, formulaic, and error-prone fashion.

To use AutoValue, create an abstract class with an abstract getter for each desired field. For example, in the `ArticleViewFragment` concrete class we might define an abstract class named `ToolbarViewData` that is annotated with `@AutoValue`:

```java
@AutoValue
static abstract class ToolbarViewData {
  abstract Article article();
  abstract ContentItemThumbnailData thumbnailData();

  public static ToolbarViewData create(final Article article,
                                       final ContentItemThumbnailData thumbnailData) {
    return new AutoValue_ArticleViewFragment_ToolbarViewData(article, thumbnailData);
  }
}
```

Clients would create a new `ToolbarViewData` implementation by invoking the `create` static factory method. This, in turn, creates and returns an instance of the generated `AutoValue_ArticleViewFragment_ToolbarViewData` subclass.

Upon compiling, the annotation processor for AutoValue creates a concrete subclass named `AutoValue_ArticleViewFragment_ToolbarViewData`. This subclass accepts an `Article` and `ContentItemThumbnailData` into its constructor, and implements the abstract getter methods accordingly. But most importantly, it implements the `equals` and `hashCode` methods appropriately:

```java
@Override
public boolean equals(Object o) {
  if (o == this) {
    return true;
  }
  if (o instanceof ArticleViewFragment.ToolbarViewData) {
    ArticleViewFragment.ToolbarViewData that = (ArticleViewFragment.ToolbarViewData) o;
    return (this.article.equals(that.article()))
        && (this.thumbnailData.equals(that.thumbnailData()));
  }
  return false;
}

@Override
public int hashCode() {
  int h = 1;
  h *= 1000003;
  h ^= article.hashCode();
  h *= 1000003;
  h ^= thumbnailData.hashCode();
  return h;
}
```

Moreover, it also implements `toString`, which facilitates with logging such values:

```java
@Override
public String toString() {
  return "ToolbarViewData{"
      + "article=" + article + ", "
      + "thumbnailData=" + thumbnailData
      + "}";
}
```

Note that its constructor also asserts that its parameters are not `null`:

```java
AutoValue_ArticleViewFragment_ToolbarViewData(
    Article article,
    ContentItemThumbnailData thumbnailData) {
  if (article == null) {
    throw new NullPointerException("Null article");
  }
  this.article = article;
  if (thumbnailData == null) {
    throw new NullPointerException("Null thumbnailData");
  }
  this.thumbnailData = thumbnailData;
}
```

To relax this restriction for a constructor parameter, we annotate the type of its corresponding getter with `@Nullable`. But we prefer `Optional` types over `@Nullable` ones. Consequently, we prefer a required `Optional` property over a `@Nullable` one.

Finally, you should always prefer a class defined by AutoValue and containing two fields over an instance of `Pair`. Such an instance can be freely used in a standard Java library. Moreover, the class name is far more self documenting, as evidenced by the `ToolbarViewData` class above.

## Understand `app` versus `core`

TODO

## Use Dagger for injection

We use [version 2 of Dagger](http://google.github.io/dagger/) to inject dependencies into components. Dependencies belonging to a similar functional unit are grouped together in a *module*. That module defines one or more *provider methods* that return objects for satisfying dependencies.

A component annotates a dependency using the `@Inject` annotation, while a provider method is annotated using a `@Provides` annotation.

For example, the `RecentlyWorkedOnModule` is a module that defines application level components for the *recently worked on* feature. This surfaces videos that the user has recently watched, or articles that the user has recently read:

```java
public class RecentlyWorkedOnModule {
  @Provides
  @Singleton
  public RecentlyWorkedOnDatabase recentlyWorkedOnDatabase(final Context context) {
    return new RecentlyWorkedOnDatabaseFactory.fromContext(context);
  }

  @Provides
  @Singleton
  public RecentlyWorkedOnManager recentlyWorkedOnManager(
      final RecentlyWorkedOnDatabase recentlyWorkedOnDatabase,
      final ObservableContentDatabase contentDatabase) {
    return new RecentlyWorkedOnManager(
        recentlyWorkedOnDatabase,
        contentDatabase,
        ObservableUtils.singleThreadScheduler(RecentlyWorkedOnManager.class.getName()),
    );
  }
}
```

The first method `recentlyWorkedOnDatabase` is a provider method that returns a `RecentlyWorkedOnDatabase` instance. The factory method for creating it requires a `Context` parameter. The `Context` instance is provided by another module named `ApplicationModule`.

The second method `recentlyWorkedOnManager` is a provider method that returns a `RecentlyWorkedOnManager` instance. The constructor for `RecentlyWorkedOnManager` requires three parameters: A `RecentlyWorkedOnDatabase` instance, a `ObservableContentDatabase` instance, and an RxJava `Scheduler` instance. The first two constructor parameters come from the parameters of the provider method, where the `RecentlyWorkedOnDatabase` instance is provided by the `recentlyWorkedOnDatabase` method, and the `ObservableContentDatabase` is provided by another module named `ContentModule`.

Our `ApplicationComponent` aggregates all the modules that provide dependencies, and also specifies an `inject` method that specifies all components with dependencies:

```java
@Singleton
@Component(modules = {
    ContentModule.class,
    ApplicationModule.class,
    RecentlyWorkedOnModule.class,
    // ... Other modules follow here.
})
public interface ApplicationComponent {
  void inject(Application application);
  void inject(DomainSubjectListFragment fragment);
  // ... Other components with dependencies follow here.
}
```

Our `Application` instance is responsible for creating the `ApplicationComponent` on startup:

```java
public class Application extends android.app.Application {
  private ApplicationComponent mApplicationComponent;

  @Override
  public void onCreate() {
    super.onCreate();

    mApplicationComponent = DaggerApplicationComponent.builder()
        .applicationModule(new ApplicationModule(this))
        .build();

    initializeDependencies();
  }
  
  // ... Remaining code follows here.
}
```

The fragment for displaying the recently worked on items can now annotate its `RecentlyWorkedOnManager` dependency with `@Inject`:

```java
@Inject
RecentlyWorkedOnManager mRecentlyWorkedOnManager;
```

And then, in its `onAttach` method, inject all of its dependencies:

```java
@Override
public void onAttach(final Activity activity) {
  super.onAttach(activity);

  final ApplicationComponent applicationComponent =
      (Application) getActivity().getApplication()).getApplicationComponent()
  applicationComponent.inject(this);
}
```

Resist using Dagger to inject dependencies into low-level components. Instead, use Dagger to inject dependencies into high-level components such as fragments, and then thread the dependencies down the object graph through constructor parameters. Such dependencies are more amenable to testing. Moreover, Dagger is currently configured to only work in the `app` module, and not in `core`. Therefore no dependencies in `core` may depend on Dagger.

When you identify a resource that should be shared by multiple components in the application, it may be suitable for injection via Dagger. Examples of such resources in our application include:

* The `OkHttpClient` instance that is configured with a cache and the client's user agent.
* Interfaces for different databases, such as `ContentDatabase`, `BookmarkDatabase`, `RecentlyWorkedOnDatabase`, and so on.
* The `ConnectivityMonitor` that monitors for network connectivity.
* The `Picasso` instance for loading and caching images.

First see if one of the existing modules is suitable for providing the resource. If not, then creating a new module to provide the resource.

## Use Retrofit for requesting resources

We use Dagger to create and customize an OkHttp instance, but our code never uses it directly. Instead, we use the OkHttp instance to create a Retrofit client. The application then uses the Retrofit client to request remote resources. These resources are typically returned in JSON format. We specify a *type adapter* to convert that JSON to a Java type that we define. We typically define that type as an immutable value type using AutoValue.

For example, using AutoValue, we define a user as:

```java
@AutoValue
public abstract class User {
  public abstract String kaid();
  public abstract Optional<String> nickname();
  public abstract Optional<HttpUrl> avatarUrl();

  public static User create(final String kaid,
                            final @Nullable String nickname,
                            final @Nullable HttpUrl avatarUrl) {
    return new AutoValue_User(
        checkNotEmpty(kaid),
        Optional.fromNullable(nickname),
        Optional.fromNullable(avatarUrl)
    );
  }
}
```

To decode a JSON object to such a value, we define a `UserJsonDecoder` class. It has a static method named `read` that accepts a `JsonReader`, consumes its parsed tokens, and returns the constructed `User` instance:

```java
@VisibleForTesting
static final class JsonKeys {
  static final String KAID = "kaid";
  static final String NICKNAME = "nickname";
  static final String AVATAR_URL = "avatarUrl";
}

public static User read(final JsonReader reader) throws IOException {
  String kaid = null;
  String nickname = null;
  HttpUrl avatarUrl = null;

  reader.beginObject();
  while (reader.hasNext()) {
    switch (reader.nextName()) {
      case JsonKeys.KAID:
        kaid = reader.nextString();
        break;
      case JsonKeys.NICKNAME:
        nickname = JsonDecoderUtils.nextStringOrNull(reader);
        break;
      case JsonKeys.AVATAR_URL:
        avatarUrl = JsonDecoderUtils.nextUrlOrNull(reader);
        break;
      default:
        reader.skipValue();
    }
  }
  reader.endObject();

  return User.create(kaid, nickname, avatarUrl);
}
```

In the `UserJsonDecoder` class we also define a static method that returns a `TypeAdapter`. The returned `TypeAdapter` knows how to create a `User` instance from the tokens read from a `JsonReader`. It does so by delegating to the `read` method in the same class:

```java
public static TypeAdapter<User> getTypeAdapter() {
  return new ReadOnlyTypeAdapter<User>() {
    @Override
    public User read(final JsonReader in) throws IOException {
      return UserJsonDecoder.read(in);
    }
  };
}
```

Finally, when we construct the `Gson` object for use with our Retrofit client, we register this `TypeAdapter` implementation:

```java
final Gson gson = new GsonBuilder()
    .registerTypeAdapter(User.class, UserJsonDecoder.getTypeAdapter())
    // ... more type adapter registrations follow
    .create();
```

And now, if a Retrofit endpoint returns a `User` instance, then the registered `TypeAdapter` automatically decodes the returned JSON and creates a new `User` instance.

## Understand our other libraries

Notable libraries from `core`. Note that these libraries depend only on the Java, Standard Edition library:

* [Guava](https://github.com/google/guava) for a bevy of helpful Java classes and methods.
* [OkHttp](http://square.github.io/okhttp/) for making HTTP and HTTP/2 requests.
* [Retrofit](http://square.github.io/retrofit/) for making HTTP and HTTP/2 requests and decoding JSON responses.
* [RxJava](https://github.com/ReactiveX/RxJava) for functional reactive programming.
* [Signpost](https://github.com/mttkay/signpost) for OAuth signing.

Notable libraries from `app`. Note that these libraries depend on the Android platform:

* The [support library](http://developer.android.com/intl/ru/tools/support-library/index.html) for `RecyclerView`, appcompat, and more.
* [Rebound](http://facebook.github.io/rebound/) for delightful spring animations.
* [Stetho](http://facebook.github.io/stetho/) for inspecting network traffic, database contents, the view hierarchy, and executing dumpapp plugins.
* [Google Play services](https://developers.google.com/android/guides/overview) to enable native login through Google.
* [Facebook Android SDK](https://developers.facebook.com/docs/android) to enable native login through Facebook.
* [ExoPlayer](https://google.github.io/ExoPlayer/) for playback of videos.
* [Butterknife](http://jakewharton.github.io/butterknife/) for binding variables to views, drawables, and other resources.
* [Picasso](http://square.github.io/picasso/) for loading both local and remote images.
* [Calligraphy](https://github.com/chrisjenx/Calligraphy) for the use of the Proxima Nova font.

## Adding a Database Table

If you need to persist a new type of data to SQLite, there are quite a few steps you need to go through. Fortunately, they are all quite straightforward and self-contained.

### Define any table names

Create a new class that defines the names of any tables you need to create. You must also create a `SelectStatementSource` for each of them; this allows you to later execute queries against the table with the given name.

For example, the following `UserDatabaseTables` class defines a single SQLite table named `UserSession`, and creates a `SelectStatementSource` for it:

```java
final class UserDatabaseTables {
  private UserDatabaseTables() {}

  static final class Names {
    public static final String USER_SESSION = "UserSession";
  }

  static final class Sources {
    public static final SelectStatementSource USER_SESSION = SelectStatementSource.table(Names.USER_SESSION);
  }
}
```

### Define the table columns

Create a new class that defines the names of all the columns in the table. Note that here you are simply defining their names; their types are defined separately in a migration below.

For example, the following `UserDatabaseTableColumns` defines the columns for the `UserSession` table:

```java
public final class UserDatabaseTableColumns {
  private UserDatabaseTableColumns() {}

  public static final class UserSessionTable {
    public static final ResultColumn KAID = ResultColumn.withName("kaid");
    public static final ResultColumn NICKNAME = ResultColumn.withName("nickname");
    ...

    public static final List<ResultColumn> ALL_COLUMNS = ImmutableList.of(
      KAID,
      NICKNAME,
      ...
    );
  }
}
```

As seen above, it is also typically helpful to define a static `ALL_COLUMNS` field. This is so you can easily build a `SelectStatement` that selects all columns from rows that satisfy a given condition.

### Define a migration

We define a *migration* any time the schema for a given table changes. When the app is first launched on a device, the table does not exist; therefore the first migration we must perform is to actually create the table.

The following shows how the migrations defined for the user database:

```java
public static List<SqlStatement> getMigrations() {
  return ImmutableList.of(
    rawSql("CREATE TABLE " + Names.USER_SESSION + " (" +
             KAID + " TEXT PRIMARY KEY NOT NULL," +
             NICKNAME + " TEXT NULL," +
             IS_PHANTOM + " BOOLEAN NOT NULL," +
             AUTH_TOKEN_VALUE + " TEXT NOT NULL," +
             AUTH_TOKEN_SECRET + " TEXT NOT NULL," +
             IS_ACTIVE + " BOOLEAN NOT NULL DEFAULT 0" +
         ")"
    ),
    rawSql("CREATE INDEX UserSessionIsActiveIndex ON " + Names.USER_SESSION + "(" +
             IS_ACTIVE +
         ")"
    ),
    rawSql("ALTER TABLE " + Names.USER_SESSION + " ADD COLUMN " +
      AVATAR_URL + " TEXT NULL"
    )
  );
}
```

This first creates the table, and then adds an index on its `IS_ACTIVE` column. Later we decided to store the `AVATAR_URL` for a user; therefore we had to define an additional migration to add this to any existing table.

Typically you only need to define a `CREATE TABLE` migration. Also you should create any `CREATE INDEX` migrations as needed.

### Define a transformer

If you want to persist instances of `MyClass` to your new table, then we must specify how to convert between a `MyClass` instance and the columns in the table. This is what a transformer does. There are two directions to the transformation.

When writing to the database, a `EntityToDatabaseRowTransformer` implementation converts from an instance of type `T` to the corresponding column values:

```java
public interface EntityToDatabaseRowTransformer<T> {
    Map<String, Object> transformEntityIntoRow(T data);
}
```

When reading from the database, a `DatabaseRowToEntityTransformer` implementation converts from the column values to an instance of type `T`:

```java
public interface DatabaseRowToEntityTransformer<T> {
  T transformRowIntoEntity(Map<String, Object> row);
}
```

For example, the `UserSessionEntityTransformer` implements both these interfaces, thereby defining both transformations for a `UserSession`:

```java
@Override
public UserSession transformRowIntoEntity(final Map<String, Object> row) {
  final User user = new User(
      (String) row.get(KAID.toString()),
      toBoolean(row.get(IS_PHANTOM.toString())),
      (String) row.get(NICKNAME.toString()),
      toUri(row.get(AVATAR_URL.toString()))
  );

  return new UserSession(
      new OAuthAccessToken(
              (String) row.get(AUTH_TOKEN_VALUE.toString()),
              (String) row.get(AUTH_TOKEN_SECRET.toString())
      ),
      user
  );
}

@Override
public Map<String, Object> transformEntityIntoRow(final UserSession userSession) {
  final Builder<String, Object> builder = ImmutableNullableMap.builder();

  return builder
      .put(KAID.toString(), userSession.user.kaid)
      .put(NICKNAME.toString(), userSession.user.nickname.orNull())
      .put(IS_PHANTOM.toString(), fromBoolean(userSession.user.isPhantom))
      .put(AVATAR_URL.toString(), fromUriOptional(userSession.user.avatarUrl))
      .put(AUTH_TOKEN_VALUE.toString(), userSession.authToken.value)
      .put(AUTH_TOKEN_SECRET.toString(), userSession.authToken.secret)
      .build();
}
```

### Define a database client

We now have all the primitives needed to create and modify a table with values of some type `T`. To modify such a table, use the static factory methods or builders of the following classes to create corresponding SQLite statements:

* `SelectStatement`
* `InsertStatement`
* `UpdateStatement`
* `DeleteStatement`

Once you have such a statement, you can execute it by passing it to the corresponding method of a `Database`. Note that if you are using a `SelectStatement`, you must pass a `DatabaseRowToEntityTransformer` parameter to convert each selected row to an instance of its corresponding type. Likewise, if you are using an `InsertStatement` or a `UpdateStatement`, you must pass a `EntityToDatabaseRowTransformer` parameter to convert each instance to the collection of column values.

For example, in the `UserDatabase` class, the following `FETCH_ALL_SESSIONS_STATEMENT` is a query that selects all columns from all rows of the `UserSession` table:

```java
private static final SelectStatement FETCH_ALL_SESSIONS_STATEMENT = SelectStatement.selectColumns(
    UserSessionTable.ALL_COLUMNS,
    Sources.USER_SESSION
);
```

The `fetchAllUserSessions` method below executes this query against a database by invoking `fetchObjects` on its `Database` instance. Its `SESSION_TRANSFORMER` instance implements `DatabaseRowToEntityTransformer`, and thereby converts each returned row to a `UserSession` instance.

```java
public Set<UserSession> fetchAllUserSessions() {
  return ImmutableSet.copyOf(mDatabase.fetchObjects(
      FETCH_ALL_SESSIONS_STATEMENT,
      SESSION_TRANSFORMER
  ));
}
```

Similarly, the `insertOrUpdateUserSession` method below inserts a given `UserSession` by invoking `insertOrReplaceRows` on its `Database` instance. The `Names.USER_SESSION` specifies the table to insert into, the `ImmutableList.of` static factory method creates the list of `UserSession` instances to insert, and the `SESSION_TRANSFORMER` instance implements `EntityToDatabaseRowTransformer` and thereby converts each `UserSession` instance to its row values:

```java
public void insertOrUpdateUserSession(final UserSession userSession) {
  mDatabase.update(InsertStatement.insertOrReplaceRows(
      Names.USER_SESSION,
      ImmutableList.of(userSession),
      SESSION_TRANSFORMER
  ));
}
```


