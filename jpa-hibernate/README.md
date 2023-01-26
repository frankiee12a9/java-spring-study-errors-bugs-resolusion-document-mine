# JPA and Hibernates relevant errors and resolutions
## 1. org.hibernate.TransientObjectException: *object references an unsaved transient instance - save the transient instance before flushing*

### Caused 💥:
In case, you have a mapping relationship in your entities which are type of collections, and that collection has one or more items which are not present (haven't persisted yet) 
in the database. By specifying the above options you tell hibernate to save them to the database when saving their parent. For example:
You have 2 entites `Movie` and `Tag` that mapped to each other as `Many-to-Many` relatioship.

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
   
   // other fields, getters, and setters are ommited...
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
   
   // other fields, getters, and setters are ommited...
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
        // setters are ommited...
        movie.setTags(tags); // tags here, is the referenced entit
        
        Movie savedMovie = movieRepository.save(movie); // here, you try to persit parent object with referenced object 
        // setters are ommited...
        
        return createMovieResponse;
    }
```

Take a look at (1) and (2) in above code, first, you try to persist tags which is collection of referenced entity and then try to save it with parent entity 
which is `Movie`. The problem is the collection of `Tag` entity hasn't managed and saved yet in the database it's just a collection of the transient(new) objects.
So Hibernate doesn't know what to do with such action. consequently, the above exception is thrown.

### RESOLUTIONS 👉:
To resolve this problem, you can try:

__1. Adding `CascadeType`__

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
   
   // other fields, getters, and setters are ommited...
}
```
NOTE:
> This can fix this issue. However, In real world scenario, we would not be creating and saving the movie's tag on each newly created movie. 
Because it will lead to generating duplicate entry issue IN CASE WE JUST WANT TO GENERATE UNIQUE TRANSACTIONS AND REUSE EXISTING ENTRY IF IT HAS BEEN
PERSISTED IN THE DATABASE.

__2. Get the persistent object from the database__

`MovieServiceImpl.java`
```java
    @Override
    public CreateMovieResponse saveMovie(CreateMovieRequest createMovieRequest) {
        List<String> tagNames = createMovieRequest.getTags();
        List<Tag> tags = new ArrayList<>(tagNames.size());
        for (String name : tagNames) {
            Tag tag = tagRepository.findByName(name).orElse(null);
			      tag = tag == null ? tagRepository.save(new Tag(name)) : tag;  // persit if object is null 
            tags.add(tag);
        }
    
        Movie movie = new Movie();
        // setters are ommited...
        movie.setTags(tags); 
        
        Movie savedMovie = movieRepository.save(movie); 
        // setters are ommited...
        
        return createMovieResponse;
    }
```

In line `104` we need to persist object in database first, so need to make the change in

`tag = tag == null ? new Tag(name) : tag;` to `tag = tag == null ? tagRepository.save(new Tag(name)) : tag;`


References:
1. https://thorben-janssen.com/entity-lifecycle-model/
2. https://stackoverflow.com/questions/2302802/how-to-fix-the-hibernate-object-references-an-unsaved-transient-instance-save?page=1&tab=scoredesc#tab-top
3. https://www.devtalkers.com/2021/07/how-to-resolve-orghibernatetransientobj.html


