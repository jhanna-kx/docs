In Windows and Linux, you can use ODBC to connect to a non-q database from q.  
<i class="fa fa-hand-o-right"></i> [ODBC q server](odbc-qserver)


## Installation

To install, download

- [odbc.k <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/master/kx/kdb%2B/c/odbc.k) into the q directory
- the appropriate odbc.so or odbc.dll:

| q        | q/l32 | q/l64 | q/w32 | q/w64 |
|----------|-------|-------|-------|-------|
| &ge;V3.0 | [odbc.so <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/master/kx/kdb%2B/l32/odbc.so) | [odbc.so <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/master/kx/kdb%2B/l64/odbc.so) |  [odbc.dll <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/master/kx/kdb%2B/w32/odbc.dll) | [odbc.dll <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/master/kx/kdb%2B/w64/odbc.dll) |
| &le;V2.8 | [odbc.so <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/0f239d00fa89254e9447949a6272e1df13ffafd6/kx/kdb%2B/l32/odbc.so) | [odbc.so <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/0f239d00fa89254e9447949a6272e1df13ffafd6/kx/kdb%2B/l64/odbc.so) | [odbc.dll <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/0f239d00fa89254e9447949a6272e1df13ffafd6/kx/kdb%2B/w32/odbc.dll) | [odbc.dll <i class="fa fa-download"></i>](https://github.com/KxSystems/code.archive/blob/0f239d00fa89254e9447949a6272e1df13ffafd6/kx/kdb%2B/w64/odbc.dll) |

!!! warning "Mixed versions"
    If you mix up the library versions, you’ll likely observe a type error when opening the connection.

Start q and load odbc.k – this populates the `.odbc` context.

!!! tip "Unix systems"
    Ensure you have <a target="_blank" href="http://www.unixodbc.com">unixODBC <i class="fa fa-external-link"></i></a> installed, 
    and that `LD_LIBRARY_PATH` includes the path to the odbc.so, e.g. for 64-bit Linux
    ```bash
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$QHOME/l64
    ```
    <i class="fa fa-hand-o-right"></i> <a target="_blank" href="http://www.easysoft.com/developer/interfaces/odbc/linux.html">unixODBC configuration guide</a> 

## Method

First open an ODBC connection to a database. 
To do so, define a <a target="_blank" href="http://en.wikipedia.org/wiki/Database_Source_Name">DSN <i class="fa 
fa-external-link"></i></a> (database source name), and then connect to the DSN using `.odbc.open`. 
This returns a connection handle, which is used for subsequent ODBC calls:
```q
q)\l odbc.k
q)h:.odbc.open `northwind               / open northwind database
q).odbc.tables h                        / list tables
`Categories`Customers`Employees`Order Details`Orders`Products..
q).odbc.eval[h;"select * from Orders"]  / run a select statement
OrderID CustomerID EmployeeID OrderDate  RequiredDate..
-----------------------------------------------------..
10248   WILMK      5          1996.07.04 1996.08.01  ..
10249   TRADH      6          1996.07.05 1996.08.16  ..
10250   HANAR      4          1996.07.08 1996.08.05  ..
..
```
Alternatively, use `.odbc.load` to load the entire database into q:
```q
q)\l odbc.k
q).odbc.load `northwind                 / load northwind database
q)Orders
OrderID| CustomerID EmployeeID OrderDate  RequiredDate ..
-------| ----------------------------------------------..
10248  | WILMK      5          1996.07.04 1996.08.01   ..
10249  | TRADH      6          1996.07.05 1996.08.16   ..
10250  | HANAR      4          1996.07.08 1996.08.05   ..
..
```


## ODBC functions

<!-- WTF?
```
#!comment
[#fkey fkey], [#fkeys fkeys], [#keys keys], [[#skey skey], [#xfkey xfkey]
```
-->
Functions defined in the `.odbc` context:


### `close`

Closes an ODBC connection handle:
```q
q).odbc.close h
```


### `eval`

Evaluate a SQL expression:
```q
q)sel:"select CompanyName,Phone from Customers where City='London'"
q)b:.odbc.eval[h;sel]
q)b
CompanyName             Phone
----------------------------------------
"Around the Horn"       "(171) 555-7788"
"B's Beverages"         "(171) 555-1212"
"Consolidated Holdings" "(171) 555-2282"
"Eastern Connection"    "(171) 555-0297"
"North/South"           "(171) 555-7733"
"Seven Seas Imports"    "(171) 555-1717"
q)select from b where Phone like "*1?1?"
CompanyName          Phone
-------------------------------------
"B's Beverages"      "(171) 555-1212"
"Seven Seas Imports" "(171) 555-1717"
```


### `load`

Loads an entire database into the session:
```q
q).odbc.load `northwind
q)\a
`Categories`Customers`Employees`OrderDetails`Orders`Products`Shippers`Supplie..
q)Shippers
ShipperID| CompanyName        Phone
---------| -----------------------------------
1        | "Speedy Express"   "(503) 555-9831"
2        | "United Package"   "(503) 555-3199"
3        | "Federal Shipping" "(503) 555-9931"
```


### `open`

Open a connection to a database, returning an ODBC connection handle. For example:
```q
q)h:.odbc.open `northwind
q)h
77932560j
```


### `tables`

List tables in database:
```q
q).odbc.tables h
`Categories`Customers`Employees`Order Details`Orders`Products...
```


### `views`

List views in database:
```
q).odbc.views h
`Alphabetical List of Products`Category Sales for 1997`Current...
```


## Tracing

ODBC has the capability to trace the ODBC API calls to a log file; 
sometimes this can be helpful in resolving unusual or erroneous behaviour. 
On Unix, you can activate the tracing by adding
```
[ODBC]
Trace         = 1
TraceFile     =/tmp/odbc.log
```
to the odbcinst.ini file, which can typically be found in /etc or /usr/local/etc.

<i class="fa fa-hand-o-right"></i> <a target="_blank" href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms711034(v=vs.85).aspx">MSDN <i class="fa fa-external-link"></i></a> 
for tracing on Windows