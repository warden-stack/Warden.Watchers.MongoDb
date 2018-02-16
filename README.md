# Warden MongoDb Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.MongoDb.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.MongoDb)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.MongoDb.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.MongoDb/branches)

**MongoDbWatcher** can be used either for simple database monitoring (e.g. checking if a connection can be made) or more advanced one which may include running a specialized query.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.MongoDb)**. 
```
dotnet add package Warden.Watchers.MongoDb
```

### Configuration:

 - **WithQuery()** - executes the specified query on a selected database.
 - **WithTimeout()** - timeout after which the invalid result will be returned.
 - **EnsureThat()** - predicate containing a received query result of type *IEnumerable<dynamic>* that has to be met in order to return a valid result.
 - **EnsureThatAsync()** - async  - predicate containing a received query result of type *IEnumerable<dynamic>* that has to be met in order to return a valid result
 - **WithConnectionProvider()** - provide a  custom *IMongoDbConnection* which is responsible for making a connection to the database. 
 - **WithMongoDbProvider()** - provide a  custom *IMongoDb* which is responsible for executing a query on a database. 

**MongoDbWatcher** can be configured by using the **MongoDbWatcherWatcherConfiguration** class or via the lambda expression passed to a specialized constructor. 

Please note that either *WithConnectionProvider()* or *WithDatabaseProvider()* methods can be used to achieve the same goal, which is getting the *IMongoDb* instance. 

Example of configuring the watcher via provided configuration class:
```csharp
var configuration = MongoDbWatcherConfiguration
    .Create("MyDatabase", "mongodb://localhost:27017", timeout: TimeSpan.FromSeconds(3))
    .WithQuery("Users", "{\"name\": \"admin\"}")
    .EnsureThat(users => users.Any(user => user.role == "admin"))
    .Build();
var mongoDbWatcher = MongoDbWatcher.Create("My MongoDB watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(mongoDbWatcher)
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .AddMongoDbWatcher("mongodb://localhost:27017", "MyDatabase", cfg =>
    {
        cfg.WithQuery("Users", "{\"name\": \"admin\"}")
           .EnsureThat(users => users.Any(user => user.role == "admin"));
    }, timeout: TimeSpan.FromSeconds(3))
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**MongoDbWatcher** provides a custom **MongoDbWatcherCheckResult** type which contains additional values.

```csharp
public class MongoDbWatcherCheckResult : WatcherCheckResult
{
    public string Database { get; }
    public string ConnectionString { get; }
    public string Query { get; }
    public IEnumerable<dynamic> QueryResult { get; }
}
```

### Custom interfaces:
```csharp
public interface IMongoDbConnection
{
    string Database { get; }
    string ConnectionString { get; }
    TimeSpan Timeout { get; }
    Task<IMongoDb> GetDatabaseAsync();
}
```

**IMongoDbConnection** is responsible for making a connection to the database. It can be configured via the *WithConnectionProvider()* method. By default it is based on the **[MongoDB Driver](https://docs.mongodb.org/ecosystem/drivers/csharp/)**.

```csharp
public interface IMongoDb
{
    Task<IEnumerable<dynamic>> QueryAsync(string collection, string query);
}
```

**IMongoDb** is responsible for executing the query on a database. It can be configured via the *WithMongoDbProvider()* method. By default it is based on the **[MongoDB Driver](https://docs.mongodb.org/ecosystem/drivers/csharp/)**.