# JPA and Hibernates learning notes and takeaway, belong with relevant errors and its resolutions

## Contents

1. [org.hibernate.TransientObjectException: Object references an unsaved transient instance save the transient instance before flushing](#org-hibernate-TransientObjectException:-object-references-an-unsaved-transient-instance-save-the-transient-instance-before-flushing)
1. [Hibernate: a collection with cascade=”all-delete-orphan” was no longer referenced by the owning entity instance hibernate]
   (#Hibernate-a-collection-with-cascade=”all-delete-orphan”-was-no longer-referenced-by-the-owning-entity-instance-hibernate)
1. [this is jus a test anchor link](#this-is-jus-a-test-anchor-link)

## org.hibernate.TransientObjectException: Object references an unsaved transient instance save the transient instance before flushing

### Caused 💥:

In case, you have a mapping relationship in your entities which are type of collections, and that collection has one or more items which are not present (haven't persisted yet)
in the database. By specifying the above options you tell hibernate to save them to the database when saving their parent. For example:
You have 2 entities `Movie` and `Tag` that mapped to each other as `Many-to-Many` relatioship.

`Tag.java`

```java
@Entity
class Tag {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   private long id;

   @ManyToMany(fetch = FetchType.EAGER)
      @JoinTable(name = "movie_tag", joinColumns = @JoinColumn(name = "tag_id", referencedColumnName = "id"),
        inverseJoinColumns = @JoinColumn(name = "movie_id", referencedColumnName = "id"))
    private List<Movie> movies;

   // other fields, getters, and setters are omitted...
}
```

`Post.java`

```java
class Movie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @ManyToMany(fetch = FetchType.LAZY)
      @JoinTable(name = "movie_tag", joinColumns = @JoinColumn(name = "movie_id", referencedColumnName = "id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id", referencedColumnName = "id"))
     private List<Tag> tags;

     // other fields, getters, and setters are omitted...
}
```

The business logic requires that whenever you create a `Movie` new `Tag` is inserted and created also. For example

`MovieServiceImpl.java`

```java
    @Override
    public CreateMovieResponse saveMovie(CreateMovieRequest createMovieRequest) {
        List<String> tagNames = createMovieRequest.getTags();
        List<Tag> tags = new ArrayList<>(tagNames.size());
        for (String name : tagNames) {
            Tag tag = tagRepository.findByName(name).orElse(null);
            tag = tag == null ? new Tag(name) : tag;
            tags.add(tag);
        }

        Movie movie = new Movie();
        // setters are omitted...
        movie.setTags(tags); // (1) tags here, which are the referenced entity

        Movie savedMovie = movieRepository.save(movie); // (2) you try to persist parent object with referenced object
        // setters are omitted...

        return createMovieResponse;
    }
```

Take a look at (1) and (2) in above code, first, you try to persist tags which is collection of referenced entity and then try to save it with parent entity
which is `Movie`. The problem is the collection of `Tag` entity hasn't managed and saved yet in the database it's just a collection of the transient(new) objects.
So Hibernate doesn't know what to do with such action. consequently, the above exception is thrown.

### RESOLUTIONS 👉:

To resolve this problem, you can try:

**1. Adding `CascadeType`**

`Post.java`

```java
class Movie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @ManyToMany(fetch = FetchType.LAZY, cascade = {
        CascadeType.PERSIST,
        CascadeType.MERGE
    })
	  @JoinTable(name = "movie_tag", joinColumns = @JoinColumn(name = "movie_id", referencedColumnName = "id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id", referencedColumnName = "id"))
	  private List<Tag> tags;

   // other fields, getters, and setters are omitted...
}
```

NOTE:

> This can fix this issue. However, In real world scenario, we would not be creating and saving the movie's tag on each newly created movie.
> Because it will lead to generating duplicate entry issue IN CASE WE JUST WANT TO GENERATE UNIQUE TRANSACTIONS AND REUSE EXISTING ENTRY IF IT HAS BEEN
> PERSISTED IN THE DATABASE.

**2. Get the persistent object from the database**

`MovieServiceImpl.java`

```java
    @Override
    public CreateMovieResponse saveMovie(CreateMovieRequest createMovieRequest) {
        List<String> tagNames = createMovieRequest.getTags();
        List<Tag> tags = new ArrayList<>(tagNames.size());
        for (String name : tagNames) {
            Tag tag = tagRepository.findByName(name).orElse(null);
            tag = tag == null ? tagRepository.save(new Tag(name)) : tag;  // persist entity instance if it's null
            tags.add(tag);
        }

        Movie movie = new Movie();
        // setters are omitted...
        movie.setTags(tags);

        Movie savedMovie = movieRepository.save(movie);
        // setters are omitted...

        return createMovieResponse;
    }
```

We updated `tag = tag == null ? new Tag(name) : tag;` to `tag = tag == null ? tagRepository.save(new Tag(name)) : tag;`
as it's needed to be persisted and managed object in database first before taking any operation on it.

## Collection with cascade=”all-delete-orphan” was no longer referenced by the owning entity instance-hibernate

>

## this is jus a test anchor link

> > References:

1. https://thorben-janssen.com/entity-lifecycle-model/
2. https://stackoverflow.com/questions/2302802/how-to-fix-the-hibernate-object-references-an-unsaved-transient-instance-save?page=1&tab=scoredesc#tab-top
3. https://www.devtalkers.com/2021/07/how-to-resolve-orghibernatetransientobj.html
