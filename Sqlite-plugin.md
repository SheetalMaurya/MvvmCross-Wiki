# Requirements

## Console (Windows)

On windows you need sqlite3.dll, get it from http://www.sqlite.org/download.html (Search down for sqlite-dll-win32-XXXXXX.zip).

Once you download the .zip, extract the contained sqllite3.dll and place the file under {YourConsoleOrTestProject}/bin/Debug on the filesystem. This will resolve the "Unable to load DLL 'sqlite3': The specified module could not be found. (Exception from HRESULT: 0x8007007E)" error.

See the following link for more info:
praeclarum/sqlite-net#206

# Usage

Have a class inheriting IMvxServiceConsumer<ISQLiteConnectionFactory>
Then simple call:
````
var m = this.GetService();
ISQLiteConnection Sqlite = m.Create("mydatabasefile");
````
Sqlite is an object implementing SQLiteConnection from sqlite-net, so go [Check out their documentation for usage.](https://github.com/praeclarum/sqlite-net)