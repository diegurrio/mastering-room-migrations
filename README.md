# Room Migration Cheat Sheet

Database migrations in Room require developers to write SQL statements manually that are executed in the database. This can be tricky, since Room abstracts most of the SQL statements away from developers and we don't have to write them all the time. This cheat sheet serves as a reminder of the SQL required for various migrations.

# Types Of Migrations

There are a number of changes to your database that could require a migration between versions. We can have one or more of these changes in a given migration:

1. [Adding a field to an existing table.](#Adding-A-Field-To-An-Existing-Table)
2. [Removing a field from an existing table.](#Removing-A-Field-From-An-Existing-Table)
3. [Changing the data type or column name of a field from an existing table.](#Changing-The-Data-Type-Of-A-Field)
4. [Adding a new table to the database.](#Adding-A-New-Entity)
5. [Removing a table from the database.](#Removing-An-Entity)

If you have found yourself writing a database migration that's not in the above list, create an Issue and/or Pull Request so we can get it added to the list. :)

# Understanding Table And Column Names

It's important to remember how Room entities map to the table name and column names used in the database.

If we have an entity like this:

```kotlin
@Entity
data class Student(
    val id: Long = 0L,
    val firstName: String = ""
)
```

Then we have a table named `Student`, with two columns named `id` and `firstName`. It maps directly from class and property names. If you want granular control over this, you can add those properties inside the annotations:

```kotlin
@Entity(tableName = "student_table")
data class Student(
    val id: Long = 0L,
    @ColumnInfo(name = "first_name")
    val firstName: String = ""
)
```

# Adding A Field To An Existing Table

To add a new column, we can use the `ALTER TABLE` command:

```kotlin
database.execSQL("ALTER TABLE table_name ADD COLUMN columnName TYPE [NULL|NOT NULL] [DEFAULT default]")
```

A default is required if your new type is not nullable:

```kotlin
database.execSQL("ALTER TABLE Student ADD COLUMN age INTEGER NOT NULL DEFAULT 0")
```

If your new type is nullable, a default is not required:

```kotlin
database.execSQL("ALTER TABLE Student ADD COLUMN nickName TEXT NULL")
```

# Removing A Field From An Existing Table

SQLite does not support removing a column from a table directly. Instead we need to do the following:

1. Create a backup table that has the schema we actually want.
2. Copy everything from the original table into the backup.
3. Delete the original table.
4. Rename the backup table to the same name as the original.

Here is an example of this type of change:

```kotlin
database.execSQL("CREATE TABLE Student_backup (id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, firstName TEXT NOT NULL, age INTEGER NOT NULL)")
database.execSQL("INSERT INTO Student_backup (id, firstName, age) SELECT id, firstName, age FROM Student")
database.execSQL("DROP TABLE Student")
database.execSQL("ALTER TABLE Student_backup RENAME TO Student")
```

# Changing The Data Type Of A Field

Similar to the removing a field example, SQLite doesn't support the action of changing a data type. Instead, we need to create a back up table, copy everything over, and rename it.

Here is an example of this change:

```kotlin
database.execSQL("CREATE TABLE Student_backup (id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, firstName TEXT NOT NULL, age REAL NOT NULL)")
database.execSQL("INSERT INTO Student_backup (id, firstName, age) SELECT id, firstName, age FROM Student")
database.execSQL("DROP TABLE Student")
database.execSQL("ALTER TABLE Student_backup RENAME TO Student")
```

# More Information
This project was based on this [lecture](https://androidessence.com/mastering-room-database-migrations)
