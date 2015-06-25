## Prefer Immutability

Our principal goal as programmers is to reduce complexity. At any point in a program, we can reason about the state of a constant trivially -- its state is the state upon assignment. By contrast, the state of a variable can change endlessly.

With judicious use, immutable objects can lead to quicker execution speed and lower memory usage. The hash value of a `String`, the query parameters of an immutable URL, and the distance traced by an immutable sequence of points are all immutable as well. We can cache their values for later use instead of recompute them on every access. We can also share an immutable object with clients without requiring those clients to defensively copy it.

An immutable value is safe in many ways. For example, `String` is immutable and so its `hashCode` value is immutable. Consequently, it is safe to use as a `String` as a key in a `Map`: The lower order bits of that hash will never change, and so an inserted key will never reside in the wrong bucket. An immutable value is also thread-safe, because a client must acquire a lock only to read data that another client can concurrently modify. An immutable value renders that moot.

A good approach for creating immutable types is to define types that are simply data, and which do not have behavior. For example, consider the `SecondsWatched` class, which tracks the seconds watched values in a video:

```java
public class SecondsWatched {
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
public final class VideoUserProgress extends ContentItemUserProgress {
    public final SecondsWatched secondsWatched;
    public final Optional<Date> lastWatchedDate;

    /* Constructor, equals, hashCode, and toString follow here. */
}
```

Again, `SecondsWatched` is immutable. So is an instance of the `Date` class. A constructed `Optional` cannot be reassigned to refer to another (possibly `null`) value, and so it is immutable as well. By virtue of the `VideoUserProgress` composing instances of immutable classes, and making those fields `final`, then each `VideoUserProgress` instance is immutable as well.

Note that for such immutable classes, we do not provide getter methods for each field. Instead, these fields are have `public` visibility. This is because getter methods are typically paired with corresponding setter methods for access control, but immutable values can by definition not be set.

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

## Use Google Guava

From the Google Guava web page:

> The Guava project contains several of Google's core libraries that we rely on in our Java-based projects: collections, caching, primitives support, concurrency libraries, common annotations, string processing, I/O, and so forth.

Some of the packages are indispensable for disciplined Java development:

### Preconditions

The `Preconditions` class is found in the `com.google.common.base` package. Use it in every constructor you define to ensure that a client is constructing a valid instance. By catching such invalid data when the instance is created, we can determine the source of the invalid parameters by navigating the stacktrace. If we checked for validity upon use, we not only put the burden on every client to ensure that the data is valid, and this precludes finding where the invalid data comes from.

#### Method `checkNotNull`

The `checkNotNull` method accepts a parameter and throws an `NullPointerException` if it is `null`. It returns that parameter if it is not. Use it for objects:

```java
public final class ApiClient {
    public final ContentApi contentApi;
    public final UserApi userApi;

    public ApiClient(ContentApi contentApi, UserApi userApi) {
        this.contentApi = Preconditions.checkNotNull(contentApi);
        this.userApi = Preconditions.checkNotNull(userApi);
    }
}
```

Therefore a client of an `ApiClient` instance can be ensured that it has `contentApi` and `userApi` fields that are not `null`. It does not need to defend itself against these conditions.

#### Method `checkArgument`

The `checkArgument` method asserts that its expression is `true`. If not, it throws an `IllegalArgumentException`. It's best to provide a second parameter that provides a more detailed message for the `IllegalArgumentException`. This message should include any parameter in the expression that evaluated as `false`. This makes debugging easier. For example: 

```java
public final class SecondsWatched {
    public final long lastSecondWatched;
    public final long totalSecondsWatched;
  

    public SecondsWatched(long lastSecondWatched, long totalSecondsWatched) {
        Preconditions.checkArgument(lastSecondWatched >= 0,
                "lastSecondWatched cannot be negative: " + lastSecondWatched);
        Preconditions.checkArgument(totalSecondsWatched >= 0,
                "totalSecondsWatched cannot be negative: " + totalSecondsWatched);

        this.lastSecondWatched = lastSecondWatched;
        this.totalSecondsWatched = totalSecondsWatched;
    }
}
```

### Object utilities

For immutable objects that are just data and define no behavior, it's useful to implement `equals`, `hashCode`, and `toString`.

#### Implementing `equals`

The template for implementing `equals` is:

```java
public final class VideoSubtitle {
    public final long timeMillis;
    public final String text;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof VideoSubtitle)) return false;

        VideoSubtitle that = (VideoSubtitle) o;

        return (timeMillis == that.timeMillis
                && text.equals(that.text));
    }
}
```

As shown above, do not forget to use the `equals` method to compare instance fields that are objects (i.e. extend `Object`). Accidentally using `==` will test for identity, or whether both operands refer to the same object (i.e. the same instance at the same address in memory).

Defining an `equals` method is especially useful for testing, as it allow us to leverage the `assertEquals` method:

```
long expectedTimeMillis = 123;
String expectedText = "Let's count to three."
VideoSubtitle expectedSubtitle =
        new VideoSubtitle(expectedTimeMillis, expectedText);

assertEquals(expectedSubtitle, actualSubtitle);
```

If these values are not equal, then `assertEquals` prints both arguments in the thrown assertion. (Which is only helpful if `toString` is implemented, as described below!)

#### Implementing `equals` with inheritance

If there is a base class, then the derived class `equals` implementation should call the base class `equals` implementation. For example, consider the abstract base class `ContentItemUserProgress`:

```java
public abstract class ContentItemUserProgress {
    public final ContentItemIdentifier contentItemIdentifier;
    public final UserProgressLevel progressLevel;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ContentItemUserProgress)) return false;

        ContentItemUserProgress that = (ContentItemUserProgress) o;

        return (contentItemIdentifier.equals(that.contentItemIdentifier) &&
                progressLevel == that.progressLevel);
    }
}
```

The subclass `VideoUserProgress` tests that the instance fields in the base class are equal before testing that its own instance fields are equal:

```java
public final class VideoUserProgress extends ContentItemUserProgress {
    public final SecondsWatched secondsWatched;
    public final Optional<Date> lastWatchedDate;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof VideoUserProgress)) return false;

        if (!super.equals(o)) return false;

        VideoUserProgress that = (VideoUserProgress) o;

        return (secondsWatched.equals(that.secondsWatched) &&
                lastWatchedDate.equals(that.lastWatchedDate));
    }
}
```

#### Implementing `hashCode`

The `Objects` method from the `com.google.common.base` package contains a `hashCode` method that computes a hash value for a list of arguments. Use it to compute the hash code of all the instance fields of a class:

```
@Override
public int hashCode() {
    return Objects.hashCode(lastSecondWatched, totalSecondsWatched);
}
```

#### Implementing `hashCode` with inheritance

If there is a base class, then the derived class `hashCode` implementation should call the base class `hashCode` implementation. For example, again consider the base class `ContentItemUserProgress`:

```java
public final class VideoUserProgress extends ContentItemUserProgress {
    public final SecondsWatched secondsWatched;
    public final Optional<Date> lastWatchedDate;

    @Override
    public int hashCode() {
        return Objects.hashCode(contentItemIdentifier, progressLevel);
    }
}
```

The subclass `VideoUserProgress` mixes in the superclass `hashCode` value into the `hashCode` value that it returns:

```java
public final class VideoUserProgress extends ContentItemUserProgress {
    public final SecondsWatched secondsWatched;
    public final Optional<Date> lastWatchedDate;

    @Override
    public int hashCode() {
        return Objects.hashCode(super.hashCode(), secondsWatched, lastWatchedDate);
    }
}
```

#### Implementing `toString`

The `MoreObjects` class from the `com.google.common.base` package has a useful `toStringHelper` method that, as its name implies, helps construct a `toString` value. Again, consider the `SecondsWatched` class:

```java
public class SecondsWatched {
    public final long lastSecondWatched;
    public final long totalSecondsWatched;

    @Override
    public String toString() {
        return MoreObjects.toStringHelper(this)
                .add("lastSecondWatched", lastSecondWatched)
                .add("totalSecondsWatched", totalSecondsWatched)
                .toString();
    }
}
```

If `SecondsWatched` is instantiated with a `lastSecondWatched` value of `5` and a `totalSecondsWatched` value of `10`, then invoking its `toString` method will return:

```
SecondsWatched={lastSecondWatched=5, totalSecondsWatched=10}
```

#### Implementing `toString` with inheritance

If there is a base class, it may be helpful for it to provide a `protected` factory method that creates a `ToStringHelper` instance and adds to it the base class fields. The derived class implementation of `toString` can then invoke this base class factory method and add to it the derived class fields before returning. For example, again consider the base class `ContentItemUserProgress`:

```java
public abstract class ContentItemUserProgress {
    public final ContentItemIdentifier contentItemIdentifier;
    public final UserProgressLevel progressLevel;

    protected MoreObjects.ToStringHelper getToStringHelper() {
        return MoreObjects.toStringHelper(this)
                .add("contentItemIdentifier", contentItemIdentifier)
                .add("kind", getItemKind())
                .add("progressLevel", progressLevel);
    }
}
```

The subclass `VideoUserProgress` then adds its own fields to the `ToStringHelper` before returning:

```java
public final class VideoUserProgress extends ContentItemUserProgress {
    public final SecondsWatched secondsWatched;
    public final Optional<Date> lastWatchedDate;

    @Override
    public String toString() {
        return getToStringHelper()
                .add("secondsWatched", secondsWatched)
                .add("lastWatchedDate", lastWatchedDate)
                .toString();
    }
}
```

### Immutable Collections

As described earlier, immutability is a critical aspect of controlling complexity in our programs. As such, for each of the collection interfaces `List`, `Map`, `SortedMap`, `Set`, and `SortedSet`, Guava defines an implementation whose instances are immutable. This is only possible because the mutating methods of each collection interface, such as `add`, `set`, or `remove`, are defined as optional operations. The underlying implementation can choose to implement the method as specified by the interface, or throw an `UnsupportedOperationException`. The collections defined by Guava choose the latter.

> The standard libraries for some languages, such as Objective-C and Scala, instead define separate types for mutable and immutable collections.

#### Construction from individual elements

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

#### Construction from another collection

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

#### Construction with a builder

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

#### Defensive copying

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

#### Guarding aginst `null`

Finally, `ImmutableList` and `ImmutableSet` prohibit `null` elements, while `ImmutableMap` prohibits `null` keys or values. Attempting to construct such an instance will throw a `NullPointerException`. If you require an `ImmutableMap` that permits `null` values, we have an `ImmutableNullableMap` instance. But before building a `List` or `Set` analogue, thoroughly consider alternative designs.

### Using `Optional`

We have seen how `checkNotNull` ensures that instances are not mistakenly constructed with `null` values. But if a parameter value or a return value belonging to a method signature can be `null`, then we precede its type with the `@Nullable` annotation. For example:

```java
public static @Nullable URI toUri(final @Nullable String value) throws URISyntaxException {
  return (value != null) ? new URI(value) : null;
}
```

We can even annotate variable declarations:

```java
@Nullable URI downloadUrl = toUri(downloadUrlString);
```

In more modern languages, *nullability* is encoded by the type system. In Swift, for example, a type suffixed with `?` can assume `null`, while a type that stands alone cannot:

```swift
let x: String = "foo"
let y: String? = null

let z: String = null  // compilation error
```

In Java, The `Optional` class from Guava is a library-level imitation of this concept. It is not as robust and so it is not a substitute. But nonetheless it succeeds in forcing us to account for `null` values at compile-time.

For example, consider an `toIso8601` method that converts a `Date` parameter to a string in ISO 8601 format. It expects that its `Date` parameter is not `null`, and even uses `Preconditions.checkNotNull` to assert this.

A client may store the date at which a video was last watched by the user. It could convert the corresponding `Date` instance to ISO 8601 like so:

```java
Date lastWatchedDate = ...
String lastWatchedIso8601 = toIso8601(lastWatchedDate);
```

But this does not capture that `lastWatchedDate` can be `null` if the user has never watched the corresponding video before. Consequently, `toIso8601` will be invoked with a `null` parameter, which in turn will cause its `Preconditions.checkNotNull` statement to throw a `NullPointerException` at runtime.

By representing `lastWatchedDate` as an `Optional`, we can force accounting for the nullability of `lastWatchedDate` at compile-time. The client must explicitly test for the presence of a contained `Date` value before extracting it from the optional and passing it to the `toIso8601` method:

```java
Optional<Date> lastWatchedDate = ...
@Nullable String lastWatchedString = lastWatchedDate.isPresent()
    ? toIso8601(lastWatchedDate.get())
    : null;
```

### Using `Predicates` and `Functions`

TODO

### Using `Lists` and `Maps`

TODO

### Using `FluentIterable`

TODO

