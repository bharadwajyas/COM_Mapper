# COM_Mapper
A tool to create COM class/interface relationships in neo4j. It is designed to be run once on a developer system, and it will take a few hours to complete. Once completed, a user can issue cypher queries via Neo4j and get COM class/interface relationships. This is very much a quick and dirty prototype that was created to serve my needs for doing Microsoft Office OLE research. Please feel free to expand or shoot me ideas to make this better.

# Intro
COM_Mapper is a C# tool that maps out COM class/interface relationships in neo4j, enabling the user to perform custom queries on COM class implementation. COM_Mapper works by iterating through each COM class registered under HKCR\CLSID and HKLM\Software\WOW64Node\Classes\CLSID. COM_Mapper will initialize each class and attempt to call QueryInterface for every interface registered under HKLM\Software\Classes\Interface and HKCR\Interface. If the COM class supports the interface, a relationship will be created in neo4j. 

# Usage
You need to stand up a neo4j server and modify Program.cs to point to the instance IP and port as well as the credentails. These will be command line arguments on the next update. Once neo4j is up and running, run COM_Mapper.exe. You will need to babysit this process as many COM classes will produce popups and initializing some COM classes will cause the process to crash. If the process crashes, you can provide the last CLSID processed as a command line argument and COM_Mapper will resume at that CLSID. Once COM_Mapper is finished, you will have all COM classes and interfaces saved in a neo4j database  that you can then query for research.

To start:
```
.\COM_Mapper.exe
```

To continue after crashing:
```
.\COM_Mapper.exe "{00000000-0000-0000-0000-000000000000}"
```
Where the GUID is the last GUID printed to screen before crash.

# Examples
In your browser, go to the neo4j instance (usually http://127.0.0.1:7474) and connect to the database. After COM_Mapper.exe has finished registering all COM classes/interfaces, you can issue cypher queries to get information about COM classes and interfaces.

## Querying for all classes that implement a set of interfaces
```
MATCH (c:ComClass) - [:implements] -> (:ComInterface {name:"IPersistFile"})
MATCH (c:ComClass) - [:implements] -> (:ComInterface {name:"IOleObject"})
RETURN c
```
## Querying for all interfaces implemented by a class
```
MATCH (c:ComClass {clsid:"{1C82EAD9-508E-11D1-8DCF-00C04FB951F9}"}) - [:implements] -> (i:ComInterface)
RETURN c, i
```
```
MATCH (c:ComClass {ProgId:"ScriptBridge.ScriptBridge.1"}) - [:implements] -> (i:ComInterface)
RETURN c, i
```
## Querying for all COM classes implemented by a dll
```
MATCH (c:ComClass {InProcServer32:"C:\\Windows\\SysWOW64\\mshtml.dll"})
RETURN c
```
## Querying for all COM classes that have a LocalServer32
```
MATCH (c:ComClass) WHERE EXISTS (c.LocalServer32)
RETURN c
```

## FAQ
### Isn't there a better way to do this?
Probably.
### Isn't there already a project that does this and more?
Yeah, probably. I'd like to know about it though, so send me a link.
### Why did you do this?
I am researching components that can be substituted when initializing OLE objects in the Microsoft Office environment. Initially, I wanted to find all classes that implement IOleObject, IDataObject, and IPersistStorage, and OLE View didn't have that functionality. As I was continuing research, I found it very helpful to be able to run custom queries on a whim, like "Oh, I only need a class with an out of proc server and an IPersistFile implementation". This project is very niche, but hopefully it's helpful to those messing with COM or OLE.
