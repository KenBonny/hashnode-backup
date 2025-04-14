---
title: "Entity Framework many-to-many relationship"
datePublished: Mon Jul 09 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ite9000xgyes1dphhh905
slug: entity-framework-many-to-many-relationship
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380878452/MLcBRIzi1.jpeg
tags: csharp, databases, dotnetcore

---


While optimising a query, I noticed that a many-to-many relationship still used a class in between. There is a more optimised way to configure many-to-many relationships in Entity Framework.

Before, many-to-many relationships looked like:

```
[Table("dbo.Book")]
public class Book
{
  [Key]
  public int Id { get; set; }
  public virtual ICollection Authors { get; set; }
}
[Table("dbo.BookAuthor")]
public class BookAuthor
{
  [Key]
  public int BookId { get; set; }
  [Key]
  public int AuthorId { get; set; }
}
[Table("dbo.Author")]
public class Author
{
  [Key]
  public int Id { get; set; }
  public virtual ICollection Authors { get; set; }
}
```

There is a much more compact way of describing it:

```
[Table("dbo.Person")]
public class Book
{
  [Key]
  public int Id { get; set; }
  public virtual ICollection Authors { get; set; }
}
public class Author {
  [Key]
  public int Id { get; set; }
  public virtual ICollection Books { get; set; }
}
public class MyContext : DbContext
{
  public void OnModelCreating(DbModelBuilder modelBuilder)
  {
    modelBuilder.Entity<Author>()
      .HasMany<Order>(a => a.Books)
      .WithMany(b => b.Authors)
      .Map(map =>
          {
            map.MapLeftKey("BookId");
            map.MapRightKey("AuthorId");
            map.ToTable("BookAuthor");
          });
  }
}
```

The `OnModelCreating` method will tell Entity Framework to add a many-to-many relationship between the `Book` and `Author` table. In the back it will use a link table named `BookAuthor`. Now, you won't see this little table anymore. One less step to take and to make your code a bit more readable. Just make sure the `MapLeftKey` and `MapRightKey` are set correctly, otherwise an existing table will map the wrong items to each other.

Btw, thanks [Stack Overflow](https://stackoverflow.com/questions/35527175/entity-framework-database-first-many-to-many).
