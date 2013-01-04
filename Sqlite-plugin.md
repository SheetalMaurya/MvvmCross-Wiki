# Requirements

## Console (Windows)

On windows you need sqlite3.dll, get it from http://www.sqlite.org/download.html (Search down for sqlite-dll-win32-XXXXXX.zip)

# Usage

Have a class inheriting IMvxServiceConsumer<ISQLiteConnectionFactory>
Then simple call:
````
var m = this.GetService();
ISQLiteConnection Sqlite = m.Create("mydatabasefile");
````
Sqlite is an object implementing SQLiteConnection from sqlite-net, so go [Check out their documentation for usage.](https://github.com/praeclarum/sqlite-net)