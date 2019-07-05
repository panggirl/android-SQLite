# Room 相关
[Android官方ORM数据库Room技术解决方案：@Embedded内嵌对象（二）](https://blog.csdn.net/zhangphil/article/details/78621009)

[Android Room使用](https://www.jianshu.com/p/7354d5048597)

[Android—Room数据库多表查询](https://www.jianshu.com/p/f4923374885b)

```
// 复合Key
@Entity(primaryKeys = {"firstName", "lastName"})
// 设置表名tableName  @Entity(tableName = "users")
@Entity
public class User {
    @PrimaryKey
    private int uid;

    @ColumnInfo(name = "first_name")
    private String firstName;

    @ColumnInfo(name = "last_name")
    private String lastName;
    
    //不需要被存放到数据库中
    @Ignore

    // Getters and setters are ignored for brevity, 
    // but they're required for Room to work.
    //Getters和setters为了简单起见就省略了，但是对Room来说是必须的
}
```

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(User... users);

    @Delete
    void delete(User user);
}
```

```
@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```
```
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();
```

```
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build();

static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER");
    }
};
```

Room 无法直接存储 List 类型的数据，错误例子：
```
@Entity(tableName = "user")
data class User(

    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    val id: Int,

    @ColumnInfo(name = "name")
    val name: String,

    @ColumnInfo(name = "books")
    val books: List<Book>

)

data class Book(
    val bookName: String
)
```
我们存储的实体类中包含 List ，如果按照普通的方式去定义 Entity。编译的时候就会报以下错误：
```
Cannot figure out how to save this field into database. You can consider adding a type converter for it.
```
我们可以借助 @TypeConverter 去转换任意对象。例如定义一个 BookConverters
 ```
class BookConverters {

    @TypeConverter
    fun stringToObject(value: String): List<Book> {
        val listType = object : TypeToken<List<Book>>() {

        }.type
        return Gson().fromJson(value, listType)
    }

    @TypeConverter
    fun objectToString(list: List<Book>): String {
        val gson = Gson()
        return gson.toJson(list)
    }
}
```

在实体类中添加 @TypeConverters 注解
```
@Entity(tableName = "user")
@TypeConverters(BookConverters::class)
data class User(

    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    val id: Int,

    @ColumnInfo(name = "name")
    val name: String,

    @ColumnInfo(name = "books")
    val books: List<Book>

)

data class Book(
    val bookName: String
)
```
